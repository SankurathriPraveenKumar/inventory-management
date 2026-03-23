<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <!-- Budget Slider Card -->
    <div class="card budget-card">
      <div class="budget-header">
        <div class="budget-label-group">
          <span class="stat-label">{{ t('restocking.budgetLabel') }}</span>
          <span class="budget-value">${{ budget.toLocaleString() }}</span>
        </div>
        <div class="budget-range-labels">
          <span>{{ t('restocking.budgetMin') }}</span>
          <span>{{ t('restocking.budgetMax') }}</span>
        </div>
      </div>
      <input
        type="range"
        class="budget-slider"
        min="0"
        max="500000"
        step="1000"
        v-model.number="budget"
      />
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <!-- Summary stat cards -->
      <div class="stats-grid">
        <div class="stat-card info">
          <div class="stat-label">{{ t('restocking.itemsSelected', { count: recommendations.length }) }}</div>
          <div class="stat-value">{{ recommendations.length }}</div>
        </div>
        <div class="stat-card warning">
          <div class="stat-label">{{ t('restocking.totalCost') }}</div>
          <div class="stat-value">${{ totalCost.toLocaleString() }}</div>
        </div>
        <div class="stat-card" :class="budgetRemaining >= 0 ? 'success' : 'danger'">
          <div class="stat-label">{{ t('restocking.budgetRemaining') }}</div>
          <div class="stat-value">${{ budgetRemaining.toLocaleString() }}</div>
        </div>
      </div>

      <!-- Recommendations Table Card -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendationsTitle') }}</h3>
          <button
            class="btn-primary"
            :disabled="recommendations.length === 0 || orderPlacing"
            @click="placeOrder"
          >
            {{ orderPlacing ? t('common.loading') : t('restocking.placeOrder') }}
          </button>
        </div>

        <div v-if="orderSuccess" class="alert-success">{{ t('restocking.orderSuccess') }}</div>
        <div v-if="orderError" class="error">{{ t('restocking.orderError') }}</div>

        <div v-if="recommendations.length === 0" class="loading">
          {{ budget === 0 ? t('restocking.noRecommendations') : t('restocking.noMatchingData') }}
        </div>
        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.itemName') }}</th>
                <th>{{ t('restocking.table.trend') }}</th>
                <th>{{ t('restocking.table.currentDemand') }}</th>
                <th>{{ t('restocking.table.forecastedDemand') }}</th>
                <th>{{ t('restocking.table.demandGap') }}</th>
                <th>{{ t('restocking.table.unitCost') }}</th>
                <th>{{ t('restocking.table.recommendedQty') }}</th>
                <th>{{ t('restocking.table.estimatedCost') }}</th>
                <th>{{ t('restocking.table.funded') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.sku">
                <td><strong>{{ item.sku }}</strong></td>
                <td>{{ item.name }}</td>
                <td><span :class="['badge', item.trend]">{{ t(`trends.${item.trend}`) }}</span></td>
                <td>{{ item.currentDemand }}</td>
                <td>{{ item.forecastedDemand }}</td>
                <td><strong>+{{ item.demandGap }}</strong></td>
                <td>${{ item.unitCost.toFixed(2) }}</td>
                <td><strong>{{ item.recommendedQty }}</strong></td>
                <td>${{ item.estimatedCost.toLocaleString() }}</td>
                <td>
                  <span :class="['badge', item.fullyFunded ? 'success' : 'warning']">
                    {{ item.fullyFunded ? 'Full' : 'Partial' }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t } = useI18n()
    const loading = ref(true)
    const error = ref(null)
    const budget = ref(50000)
    const orderPlacing = ref(false)
    const orderSuccess = ref(false)
    const orderError = ref(false)

    const allForecasts = ref([])
    const allInventory = ref([])

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        orderSuccess.value = false
        orderError.value = false
        const [forecasts, inventory] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory()
        ])
        allForecasts.value = forecasts
        allInventory.value = inventory
      } catch (err) {
        error.value = 'Failed to load data: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Recommendations engine — reruns whenever budget or source data changes (no extra API calls)
    const recommendations = computed(() => {
      if (budget.value <= 0) return []

      // Build a SKU -> inventory item lookup for O(1) joins
      const inventoryMap = {}
      for (const item of allInventory.value) {
        inventoryMap[item.sku] = item
      }

      // Join forecasts with inventory on SKU — skip forecasts with no inventory match
      const matched = []
      for (const forecast of allForecasts.value) {
        const inv = inventoryMap[forecast.item_sku]
        if (!inv) continue

        matched.push({
          sku: forecast.item_sku,
          name: forecast.item_name,
          trend: forecast.trend,
          currentDemand: forecast.current_demand,
          forecastedDemand: forecast.forecasted_demand,
          demandGap: forecast.forecasted_demand - forecast.current_demand,
          unitCost: inv.unit_cost,
          warehouse: inv.warehouse,
          category: inv.category
        })
      }

      // Priority: increasing first, then by demandGap descending within each tier
      const trendOrder = { increasing: 0, stable: 1, decreasing: 2 }
      matched.sort((a, b) => {
        const trendDiff = (trendOrder[a.trend] ?? 1) - (trendOrder[b.trend] ?? 1)
        if (trendDiff !== 0) return trendDiff
        return b.demandGap - a.demandGap
      })

      // Greedy budget allocation
      let remainingBudget = budget.value
      const result = []

      for (const item of matched) {
        if (remainingBudget <= 0) break
        if (item.unitCost <= 0) continue

        const fullCost = item.forecastedDemand * item.unitCost
        let recommendedQty, fullyFunded

        if (fullCost <= remainingBudget) {
          recommendedQty = item.forecastedDemand
          fullyFunded = true
        } else {
          // Partial: buy as many units as the remaining budget allows
          recommendedQty = Math.floor(remainingBudget / item.unitCost)
          fullyFunded = false
        }

        if (recommendedQty <= 0) continue

        const estimatedCost = Math.round(recommendedQty * item.unitCost * 100) / 100
        remainingBudget -= estimatedCost

        result.push({ ...item, recommendedQty, estimatedCost, fullyFunded })
      }

      return result
    })

    const totalCost = computed(() =>
      Math.round(recommendations.value.reduce((sum, item) => sum + item.estimatedCost, 0) * 100) / 100
    )

    const budgetRemaining = computed(() =>
      Math.round((budget.value - totalCost.value) * 100) / 100
    )

    const placeOrder = async () => {
      if (recommendations.value.length === 0) return
      orderPlacing.value = true
      orderSuccess.value = false
      orderError.value = false

      try {
        const payload = {
          customer: 'Restocking System',
          items: recommendations.value.map(item => ({
            sku: item.sku,
            name: item.name,
            quantity: item.recommendedQty,
            unit_price: item.unitCost
          })),
          warehouse: null,
          category: null
        }
        await api.createOrder(payload)
        orderSuccess.value = true
      } catch (err) {
        console.error('Failed to place restocking order:', err)
        orderError.value = true
      } finally {
        orderPlacing.value = false
      }
    }

    onMounted(loadData)

    return {
      t,
      budget,
      loading,
      error,
      recommendations,
      totalCost,
      budgetRemaining,
      orderPlacing,
      orderSuccess,
      orderError,
      placeOrder
    }
  }
}
</script>

<style scoped>
.budget-card {
  margin-bottom: 1.5rem;
}

.budget-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 0.75rem;
}

.budget-label-group {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.budget-value {
  font-size: 2rem;
  font-weight: 700;
  color: #2563eb;
  letter-spacing: -0.025em;
}

.budget-range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #64748b;
  margin-top: 0.5rem;
  width: 100%;
}

.budget-slider {
  width: 100%;
  height: 6px;
  accent-color: #2563eb;
  cursor: pointer;
}

.btn-primary {
  background: #2563eb;
  color: white;
  border: none;
  padding: 0.625rem 1.5rem;
  border-radius: 6px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease;
}

.btn-primary:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-primary:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.alert-success {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  color: #065f46;
  padding: 0.875rem 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
  font-size: 0.938rem;
  font-weight: 500;
}
</style>
