<template>
  <div class="min-h-screen bg-gradient-to-b from-pink-50 to-pink-100 p-6 flex flex-col items-center gap-8">
    <!-- Weekly Planner -->
    <h2 class="text-2xl font-semibold mb-6 text-pink-700">💖 Weekly Meal Planner</h2>

    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-7 gap-4 w-full max-w-6xl">
      <div v-for="day in days" :key="day" class="bg-white border border-pink-100 rounded-2xl shadow-md p-4 flex flex-col space-y-3">
        <h3 class="text-lg font-semibold text-center text-pink-600">{{ day }}</h3>

        <div>
          <label class="block text-sm font-medium mb-1 text-pink-500">Lunch</label>
          <select v-model="planner[day].lunch" class="w-full border border-pink-200 rounded-xl px-3 py-2 bg-pink-50 focus:outline-none focus:ring-2 focus:ring-pink-200">
            <option value="">—</option>
            <option v-for="meal in meals" :key="meal.id + '-lunch'" :value="meal.name">{{ meal.name }}</option>
          </select>
        </div>

        <div>
          <label class="block text-sm font-medium mb-1 text-pink-500">Dinner</label>
          <select v-model="planner[day].dinner" class="w-full border border-pink-200 rounded-xl px-3 py-2 bg-pink-50 focus:outline-none focus:ring-2 focus:ring-pink-200">
            <option value="">—</option>
            <option v-for="meal in meals" :key="meal.id + '-dinner'" :value="meal.name">{{ meal.name }}</option>
          </select>
        </div>
      </div>
    </div>

    <!-- Grocery List -->
    <div class="max-w-md w-full p-6 bg-white border border-pink-100 rounded-2xl shadow-md mt-6">
      <h2 class="text-2xl font-semibold mb-4 text-pink-700">🛒 Grocery List</h2>

      <ul v-if="groceryItems.length" class="space-y-3">
        <li v-for="(item, index) in groceryItems" :key="index" class="flex items-center gap-3 p-2 rounded-lg bg-pink-50">
          <input type="checkbox" v-model="item.checked" class="w-5 h-5" />
          <span :class="{ 'line-through text-pink-300': item.checked }" class="text-pink-700">{{ item.display }}</span>
        </li>
      </ul>

      <p v-else class="text-pink-500">Select meals to generate groceries.</p>
    </div>
  </div>
</template>

<script>
import { db } from "../firebase"
import { collection, getDocs, doc, setDoc, getDoc } from "firebase/firestore"

export default {
  name: "WeeklyPlanner",
  data() {
    const days = ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"]
    return {
      days,
      meals: [],
      planner: days.reduce((acc, day) => {
        acc[day] = { lunch: "", dinner: "" }
        return acc
      }, {}),
    }
  },
  computed: {
    groceryItems() {
      if (!this.planner || !this.meals) return []

      // Use a map where key = amount, value = combined display strings with count
      const amountMap = {}

      for (const day in this.planner) {
        const dayPlan = this.planner[day]
        if (!dayPlan) continue

        ;[dayPlan.lunch, dayPlan.dinner].forEach(mealName => {
          if (!mealName) return
          const meal = this.meals.find(m => m.name === mealName)
          if (!meal || !meal.ingredients) return

          meal.ingredients.forEach(ing => {
            const name = (ing.name || "").toString().trim()
            if (!name) return
            const amount = (ing.amount === undefined || ing.amount === null) ? "" : String(ing.amount).trim()

            // Merge items based on amount
            if (amountMap[amount]) {
              // append name to existing display and increment count
              amountMap[amount].names += `, ${name}`
              amountMap[amount].count += 1
            } else {
              amountMap[amount] = { amount, names: name, count: 1, checked: false }
            }
          })
        })
      }

      // Build final display strings with count prefix
      return Object.values(amountMap).map(item => ({
        display: `${item.count} x ${item.names} ${item.amount}`.trim(),
        amount: item.amount,
        checked: item.checked
      }))
    }
  },
  methods: {
    async loadMeals() {
      try {
        const snapshot = await getDocs(collection(db, "meals"))
        this.meals = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }))
        await this.loadPlanner() // load saved planner after meals
      } catch (error) {
        console.error("Error loading meals:", error)
      }
    },

    async savePlanner() {
      try {
        await setDoc(doc(db, "planner", "default"), { planner: this.planner })
      } catch (error) {
        console.error("Error saving planner:", error)
      }
    },

    async loadPlanner() {
      try {
        const docRef = doc(db, "planner", "default")
        const docSnap = await getDoc(docRef)
        if (docSnap.exists()) {
          this.planner = docSnap.data().planner
        }
      } catch (error) {
        console.error("Error loading planner:", error)
      }
    }
  },
  mounted() {
    this.loadMeals()
  },
  watch: {
    planner: {
      handler() {
        this.savePlanner()
      },
      deep: true
    }
  }
}
</script>


