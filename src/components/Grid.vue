<template>
  <div class="navigation">
    <div class="grid">
      <!-- Grid Controls -->
      <div class="grid-controls">
        <input type="text" v-model="searchQuery" placeholder="Search deals..." @input="filterDeals" />
        <p>Selected Rows: {{ selectedRows.length }}</p>
        <button @click="exportToCSV">Export to CSV</button>
      </div>

      <!-- Grid Table -->
      <table class="grid-table">
        <thead>
          <tr>
            <th v-for="col in dynamicColumns" :key="col.key" @click="sortByColumn(col.key)">
              {{ col.label }}
              <span v-if="sortColumn === col.key">{{ sortDirection }}</span>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr 
            v-for="deal in filteredDeals" 
            :key="deal.id" 
            @click="selectDeal(deal)" 
            :class="{ selected: selectedRows.includes(deal.id) }"
          >
            <td v-for="col in dynamicColumns" :key="col.key">
              <ul v-if="Array.isArray(deal[col.key])">
                <li v-for="item in deal[col.key]" :key="item">{{ item }}</li>
              </ul>
              <span v-else>{{ deal[col.key] }}</span>
            </td>
          </tr>

        </tbody>
      </table>
    </div>

    <!-- Deal Details Panel -->
    <div class="deals-detail" v-if="selectedDeal">
      <div class="details-pane">
        <h3 class="header-title">Deal Details</h3>
        <div class="content-details">
          <!-- Dynamically generate details from the selected deal -->
          <div v-for="(value, key) in selectedDeal" :key="key">
            <p>
              <strong>{{ formatLabel(key) }}:</strong>
              <!-- If value is an array, display each item in a list -->
              <ul v-if="Array.isArray(value)">
                <li v-for="item in value" :key="item">{{ item }}</li>
              </ul>
              <!-- else, display the value directly -->
              <span v-else>{{ value }}</span>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import dealsData from '../assets/deals.json'
import exportCSV from 'json-to-csv-export'

const searchQuery = ref('')
const sortColumn = ref(null)
const sortDirection = ref(null)
const selectedDeal = ref(null)
const selectedRows = ref([])

const asc = '▼', desc = '▲';

// Extract columns dynamically from the first deal's keys
const dynamicColumns = ref([])

const filteredDeals = computed(() => {
  return dealsData.filter(deal => {
    return dynamicColumns.value.some(col => 
      String(deal[col.key]).toLowerCase().includes(searchQuery.value.toLowerCase())
    )
  }).sort((a, b) => {
    if (!sortColumn.value) return 0
    const direction = sortDirection.value === asc ? 1 : -1
    if (a[sortColumn.value] < b[sortColumn.value]) return -1 * direction
    if (a[sortColumn.value] > b[sortColumn.value]) return 1 * direction
    return 0
  })
})

/**
 * Sorts the grid by the given column key.
 * If the column is already sorted, reverses the sort order.
 * Otherwise, sorts the column in ascending order.
 * @param {string} key - the column key to sort by
 */
function sortByColumn(key) {
  if (sortColumn.value === key) {
    sortDirection.value = sortDirection.value === asc ? desc : asc
  } else {
    sortColumn.value = key
    sortDirection.value = asc
  }
}

function filterDeals() {
  // Re-computing will handle the filtering
}

function selectDeal(deal) {
  const index = selectedRows.value.indexOf(deal.id)

  if (index === -1) {
    // if the row is not selected, select it
    selectedRows.value.push(deal.id)
  } else {
    //if the row is already selected, deselect it
    selectedRows.value.splice(index, 1)
  }

  // if there is more than one row selected, close the details panel  
  if (selectedRows.value.length > 1) {
    selectedDeal.value = null
  } else if (selectedRows.value.length === 1) {
    // if only a row is selected, open the details panel. 
    selectedDeal.value = dealsData.find(deal => deal.id === selectedRows.value[0])
  } else {
    // if there are no rows selected, close the details panel. 
    selectedDeal.value = null
  }
}


/**
 * Exports the filtered deals to a CSV file named "deals.csv"
 */
function exportToCSV() {
  exportCSV({ data: filteredDeals.value, filename: 'deals.csv' })
}

// Extract column keys and labels dynamically
onMounted(() => {
  if (dealsData.length > 0) {
    const firstDeal = dealsData[0]
    dynamicColumns.value = Object.keys(firstDeal).map(key => ({
      key: key,
      label: key.replace(/_/g, ' ').toUpperCase()  // Create a readable label
    }))
  }
})

// Helper function to format labels
function formatLabel(key) {
  return key.replace(/_/g, ' ').replace(/\b\w/g, char => char.toUpperCase())
}
</script>

<style scoped lang="scss">
.navigation {
  display: flex;
  justify-content: space-between;
}

.grid-controls {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.grid-table {
  width: 100%;
  border-collapse: collapse;

  th, td {
    padding: 10px;
    border: 1px solid #ddd;
    text-align: left;
  }

  th {
    cursor: pointer;
    background-color: #007bff;
  }

  tr:hover {
    background-color: #9F5F27;
  }
}

.deals-detail {
  width: 40%;
  padding-left: 25px;
}

.details-pane {
  background-color: black;
  border-left: 1px solid #ddd;
}

.header-title {
  background-color: #007bff;
}

.content-details {
  padding: 20px;
}

.selected {
  background-color: #334968;
}
</style>


