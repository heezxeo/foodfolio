<template>
  <div v-if="showEditForm" class="edit-food-form-overlay">
    <div class="edit-food-form-content">
      <h2>Editing Food Item</h2>
      <form @submit.prevent="submitEditForm" class="edit-food-form-container">
        <div class="form-group">
          <label for="editFoodName">Food:</label>
          <input type="text" id="editFoodName" v-model="editedItem.name" required>
        </div>
        <div class="form-group">
          <label for="editFoodQuantity">Quantity:</label>
          <input type="number" id="editFoodQuantity" v-model="editedItem.quantity" required>
        </div>
        <div class="form-group">
          <label for="editFoodExpiryDate">Expiry Date:</label>
          <input type="date" id="editFoodExpiryDate" v-model="editedItem.expiryDate" required>
        </div>
        <div class="button-container"> 
          <button class="cancel-edit-button" type="button" @click="closeEditForm">Cancel</button>
          <button class="cancel-edit-button" type="submit">Update</button>
        </div>
      </form>
    </div>
  </div>
</template>
   
<script>

import { db } from '../firebase.js'; 
import { getFirestore } from 'firebase/firestore'
import { collection, doc, setDoc } from 'firebase/firestore';

export default {
  props: ['item', 'showEditForm', 'selectedCategory', 'itemToEdit', 'userEmail'], 

  computed: {
    editedItem() {
      return {
        ...this.itemToEdit, 
        id: this.itemToEdit.id, 
      }
    }
  }, 

  watch: {
    userEmail: {
    immediate: true,
      handler(newVal, oldVal) {
        console.log("userEmail in editFood:", newVal);
      }
    }
  },

  methods: {
    async submitEditForm() {
      // Assuming you have the item's ID and the updated data
      this.editedItem.category = this.selectedCategory; 
      const itemRef = doc(db, `${this.userEmail}/grocery-management/${this.selectedCategory}`, this.itemToEdit.id);

      try {
        console.log('Updating item at path:', itemRef.path);
        console.log('Updated item data:', this.editedItem);
        await setDoc(itemRef, this.editedItem, { merge: true });
        console.log('Food item updated successfully');
        this.$emit('update-item', this.editedItem);
        this.closeEditForm();
      } catch (error) {
        console.error('Error updating food item:', error);
      }
    },

    closeEditForm() {
      this.$emit('close-edit-form');
    }
  }
};
</script>
   

<style scoped>
.edit-food-form-overlay {
  display: block;
  position: fixed;
  z-index: 2;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0,0,0,0.4);
}

.edit-food-form-content {
  background-color: #fefefe; 
  margin: 15% auto; 
  padding: 20px; 
  border: 1px solid #888; 
  width: 40%; 
  border-radius: 40px; 
}

.edit-food-form-container .form-group {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 25px;
  width: 50%;
  margin-left: auto;
  margin-right: auto;
}

.form-group:hover {
  transform: scale(1.02);
}

.edit-food-form-container .form-group label {
  flex: 1;
  text-align: right;
  margin-right: 10px;
}

.edit-food-form-container .form-group input {
  flex: 2;
}

.edit-food-form-container button {
  padding: 5px 10px; 
  margin: 0 5px; 
  background-color: orange; 
  color: green; 
  border: none; 
  cursor: pointer; 
  border-radius: 20px; 
  width: auto; 
}

.edit-food-form-container .button-container {
  display: flex; 
  justify-content: center; 
  gap: 10px; 
}


.edit-food-form-content h2 {
  text-align: center; 
}


.edit-food-form-container input[type="text"],
.edit-food-form-container input[type="number"],
.edit-food-form-container input[type="date"] {
  border-radius: 20px; 
  border: 1px solid green;
  color: #a0d18c; 
  width: 100%; 
  height: 30px; 
  padding-left: 20px; 
  box-sizing: border-box; 
}


.edit-food-form-container input[type="text"]:focus,
.edit-food-form-container input[type="number"]:focus,
.edit-food-form-container input[type="date"]:focus {
  outline: none; 
  border: 1px solid #a0d18c; 
  color: green; 
}

.cancel-edit-button:hover {
  transform: scale(1.1);
  font-weight: bold;
}
</style>
