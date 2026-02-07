<template>
  <div class="max-w-md mx-auto p-6 bg-pink-50 border border-pink-100 rounded-3xl shadow-md">
    <h2 class="text-2xl font-semibold mb-4 text-pink-700">🍓 Meal Creator</h2>

    <label class="block text-sm font-medium mb-1">Meal name</label>
    <input
      v-model="mealName"
      type="text"
      placeholder="e.g. Spaghetti Bolognese"
      class="w-full mb-4 px-3 py-2 border border-pink-200 rounded-xl bg-white focus:outline-none focus:ring-2 focus:ring-pink-200"
    />

    <button
      @click="openPrompt"
      class="w-full bg-gradient-to-r from-pink-400 to-pink-300 text-white py-2 rounded-2xl hover:from-pink-500 hover:to-pink-400 transition"
    >
      ✨ Create recipe
    </button>

    <ul v-if="ingredients.length" class="mt-4 space-y-2">
      <li v-for="(item, index) in ingredients" :key="index" class="text-sm text-gray-700">
        • {{ item.name }} — {{ item.amount }}
      </li>
    </ul>

    <button
      v-if="ingredients.length"
      @click="saveMeal"
      class="mt-6 w-full bg-pink-500 text-white py-2 rounded-2xl hover:bg-pink-600 transition"
    >
      💾 Save to Firebase
    </button>
  </div>
</template>

<script>
import { db } from "../firebase"
import { collection, addDoc, serverTimestamp } from "firebase/firestore"

export default {
  name: "MealCreator",
  data() {
    return {
      mealName: "",
      ingredients: [],
    }
  },
  methods: {
    openPrompt() {
      const input = prompt(
        "Enter ingredients like this:\n500 grams:spaghetti,1: passata bottle,2: garlic cloves"
      )
      if (!input) return

      this.ingredients = input.split(",").map(item => {
        const [name, amount] = item.split(":")
        return { name: name?.trim() || "", amount: amount?.trim() || "" }
      })
    },
    async saveMeal() {
      if (!this.mealName || !this.ingredients.length) {
        alert("Please add a meal name and ingredients")
        return
      }

      try {
        await addDoc(collection(db, "meals"), {
          name: this.mealName,
          ingredients: this.ingredients,
          createdAt: serverTimestamp(),
        })

        this.mealName = ""
        this.ingredients = []
        alert("Meal saved successfully 🍝")
      } catch (error) {
        console.error(error)
        alert("Error saving meal")
      }
    },
  },
}
</script>
