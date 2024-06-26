<template>
   <NavBar />
    <div class="profile-container">
        <h1>Profile Page</h1><br>
        <div class="form-row">
            <label for="email">Email:</label>
            <div class="input-text"> {{ userEmail }} </div>
        </div>

        <div class="form-row">
            <label for="username">Username:</label>
            <div class="input-text"> {{ userName }} </div>
        </div>

        <div class="form-row">
            <label for="weight">Weight (kg):</label>
            <input v-model="userWeight" id="weight" placeholder="Enter your weight">
            <button id="weight-button" @click="saveData('weight', userWeight)">Save Weight</button>
        </div>

        <div class="form-row">
            <label for="height">Height (cm):</label>
            <input v-model="userHeight" id="height" placeholder="Enter your height">
            <button id="height-button" @click="saveData('height', userHeight)">Save Height</button>
        </div>

        <div class="form-row">
            <label for="calorieIntake">Calorie Intake Goal (kcal):</label>
            <input v-model="calorieIntakeGoal" id="calorieIntake" placeholder="Enter your calorie intake goal">
            <button id="calorie-button" @click="saveData('calorieIntakeGoal', calorieIntakeGoal)">Save Goal</button>
        </div>

        <br>
        <div class="change-password" v-if="canChangePassword">
            <button id="change-password-btn" @click="promptPasswordChange">Change Password</button>
        </div>
        <br>
    </div>


</template>

<script>
    import { getAuth, reauthenticateWithCredential, EmailAuthProvider, updatePassword, onAuthStateChanged } from 'firebase/auth';
    // Import the db here because we need firestore 
    import { db } from '@/firebase';
    import { doc, getDoc, setDoc } from 'firebase/firestore';
    import NavBar from '@/components/NavBar.vue';

    export default {
        components: {
            NavBar
        },

        data() {
            return {
            userEmail: '',
            userName: '',
            userWeight: '',
            userHeight: '',
            calorieIntakeGoal: '',
            canChangePassword: false,
            };
        },

        mounted() {
            this.fetchUserProfile();

        },

        methods : {
            fetchUserProfile() {
                const auth = getAuth();
                onAuthStateChanged(auth, (user) => {
                if (user) {
                    this.userEmail = user.email;
                    this.userName = user.displayName || user.email;
                    this.fetchAdditionalInfo(user.email);

                    // check the user's sign-in method -- if google sign-in then cannot see change pw btn
                    this.canChangePassword = user.providerData.some(
                        provider => provider.providerId === 'password'
                    );
                }
                });
            },

            async fetchAdditionalInfo(userEmail) {
                // const userDocRef = doc(db, "users", userId);
                const userInfoRef = doc(db, String(userEmail), "profile-info");
                try {
                const docSnap = await getDoc(userInfoRef);
                if (docSnap.exists()) {
                const data = docSnap.data();
                this.userWeight = data.weight || '';
                this.userHeight = data.height || '';
                this.calorieIntakeGoal = data.calorieIntakeGoal || '';
                } else {
                console.log("No data available");
                }
                } catch (error) {
                    console.error("Failed to fetch user data:", error);
                }
            },

            // This method is responsible for saving info of height,weight,kcal goal to firestore:
            async saveData(field, value) {
                const auth = getAuth();
                const userEmail = auth.currentUser.email;
                const userInfoRef = doc(db, String(userEmail), "profile-info");
                try {
                    // Set the "field" data in Firestore
                    await setDoc(userInfoRef, { [field]: value }, { merge: true });
                    alert(`${field} updated successfully.`);
                } catch (error) {
                    console.error(`Failed to update ${field}: `, error);
                    alert(`Failed to update ${field}: ${error.message}`);
                }
            },

            async promptPasswordChange() {
            // Prompt user for current and new password, probably using a modal or prompt
            // For example purposes, I'm using prompt() which is not recommended for production
            const currentPassword = prompt('Please enter your current password');
            const newPassword = prompt('Please enter your new password');
            
            if (currentPassword && newPassword) {
                try {
                const auth = getAuth();
                const user = auth.currentUser;
                const credential = EmailAuthProvider.credential(
                    user.email, // The user's email
                    currentPassword // The current password
                );

                // Reauthenticate the user
                await reauthenticateWithCredential(user, credential);
                
                // Update the password
                await updatePassword(user, newPassword);
                alert('Password updated successfully!');
                } catch (error) {
                console.error('Error updating password:', error);
                alert('Failed to update password. ' + error.message);
                }
            }
        },

    }
}
</script>

<style scoped>
.profile-container {
  display: flex;
  flex-direction: column;
  align-items: center; 
  margin: 0 auto;
  position: relative;
  top: 50px;
}

.input-text {
    width: 300px;
    padding: 7px;
    border-radius: 20px;
    border: 1.5px solid #578855; 
    font-size: 15px;
    display: flex;
    align-items: center;
}

.form-row {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  width: 100%;
  font-size: 17px;
}

.form-row label {
  flex-shrink: 0; 
  width: 230px; 
  text-align: right;
  margin-right: 10px;
}

.form-row p {
  margin: 0; 
  flex-shrink: 0; 
  width: 200px; 
  text-align: right;
}

input {
  width: 300px;
  padding: 10px;
  border-radius: 20px;
  border-color: #578855;
  background-color: #FDF0C3;
  font-family: 'Fuzzy Bubbles', cursive; 
  font-size: 15px;
  margin-right: 10px;
}

input:hover {
  transform: scale(1.02);
}

button {
  background-color: #FFB356;
  border-radius: 20px;
  color: #578855;
  padding: 5px 15px;
  cursor: pointer;
  font-family: 'Fuzzy Bubbles';
  font-size: 18px;
  transition: background-color 0.1s, transform 0.1s;
}

#change-password-btn:hover,
#weight-button:hover,
#height-button:hover,
#calorie-button:hover {
  background-color: #f8961f; 
  transform: scale(1.1); 
}

h1 {
  font-weight: bold;
}
</style>