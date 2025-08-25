<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Settings</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="showProfileModal = true">
            <ion-icon :icon="personCircle" size="large"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Settings</ion-title>
        </ion-toolbar>
      </ion-header>

      <!-- User Profile Card -->
      <div class="profile-card">
        <div class="profile-avatar" @click="showProfileModal = true">
          <ion-icon :icon="personCircle" size="large"></ion-icon>
        </div>
        <div class="profile-info">
          <h2>{{ userName }}</h2>
          <p>{{ userEmail }}</p>
          <ion-chip color="primary" outline>
            <ion-icon :icon="shield"></ion-icon>
            <ion-label>{{ subscriptionPlan }}</ion-label>
          </ion-chip>
        </div>
        <ion-button fill="clear" @click="showProfileModal = true">
          <ion-icon :icon="chevronForward"></ion-icon>
        </ion-button>
      </div>

      <!-- Security Settings -->
      <div class="settings-section">
        <ion-list-header>
          <ion-label>Security</ion-label>
        </ion-list-header>
        
        <ion-list>
          <ion-item>
            <ion-icon :icon="fingerPrint" slot="start" color="primary"></ion-icon>
            <ion-label>Biometric Authentication</ion-label>
            <ion-toggle v-model="biometricAuth" @ionChange="toggleBiometric" :enable-on-off-labels="true"></ion-toggle>
          </ion-item>

          <ion-item>
            <ion-icon :icon="time" slot="start" color="primary"></ion-icon>
            <ion-label>Auto-Lock</ion-label>
            <ion-select v-model="autoLockTime" placeholder="Select time" @ionChange="updateAutoLock">
              <ion-select-option value="0">Immediately</ion-select-option>
              <ion-select-option value="1">1 minute</ion-select-option>
              <ion-select-option value="5">5 minutes</ion-select-option>
              <ion-select-option value="15">15 minutes</ion-select-option>
              <ion-select-option value="-1">Never</ion-select-option>
            </ion-select>
          </ion-item>

          <ion-item button @click="showMasterPasswordModal = true">
            <ion-icon :icon="keyOutline" slot="start" color="primary"></ion-icon>
            <ion-label>
              <h3>Change Master Password</h3>
              <p>Update your master password</p>
            </ion-label>
            <ion-icon :icon="chevronForward" slot="end" color="medium"></ion-icon>
          </ion-item>

          <ion-item>
            <ion-icon :icon="eyeOff" slot="start" color="primary"></ion-icon>
            <ion-label>Hide Passwords by Default</ion-label>
            <ion-toggle v-model="hidePasswords" @ionChange="toggleHidePasswords" :enable-on-off-labels="true"></ion-toggle>
          </ion-item>
        </ion-list>
      </div>

      <!-- Vault Management -->
      <div class="settings-section">
        <ion-list-header>
          <ion-label>Vault</ion-label>
        </ion-list-header>
        
        <ion-list>
          <ion-item button @click="exportData">
            <ion-icon :icon="downloadOutline" slot="start" color="secondary"></ion-icon>
            <ion-label>
              <h3>Export Data</h3>
              <p>Download encrypted backup</p>
            </ion-label>
            <ion-icon :icon="chevronForward" slot="end" color="medium"></ion-icon>
          </ion-item>

          <ion-item button @click="showImportModal = true">
            <ion-icon :icon="cloudUpload" slot="start" color="secondary"></ion-icon>
            <ion-label>
              <h3>Import Data</h3>
              <p>Import from other password managers</p>
            </ion-label>
            <ion-icon :icon="chevronForward" slot="end" color="medium"></ion-icon>
          </ion-item>

          <ion-item>
            <ion-icon :icon="sync" slot="start" color="secondary"></ion-icon>
            <ion-label>Auto-Sync</ion-label>
            <ion-toggle v-model="autoSync" @ionChange="toggleAutoSync" :enable-on-off-labels="true"></ion-toggle>
          </ion-item>
        </ion-list>
      </div>

      <!-- App Preferences -->
      <div class="settings-section">
        <ion-list-header>
          <ion-label>Preferences</ion-label>
        </ion-list-header>
        
        <ion-list>
          <ion-item>
            <ion-icon :icon="moon" slot="start" color="dark"></ion-icon>
            <ion-label>Dark Mode</ion-label>
            <ion-toggle v-model="darkMode" @ionChange="toggleDarkMode" :enable-on-off-labels="true"></ion-toggle>
          </ion-item>

          <ion-item button @click="showAboutModal = true">
            <ion-icon :icon="informationCircle" slot="start" color="dark"></ion-icon>
            <ion-label>
              <h3>About</h3>
              <p>Version {{ appVersion }}</p>
            </ion-label>
            <ion-icon :icon="chevronForward" slot="end" color="medium"></ion-icon>
          </ion-item>

          <ion-item button @click="contactSupport">
            <ion-icon :icon="help" slot="start" color="dark"></ion-icon>
            <ion-label>
              <h3>Help & Support</h3>
              <p>Get help and contact us</p>
            </ion-label>
            <ion-icon :icon="chevronForward" slot="end" color="medium"></ion-icon>
          </ion-item>
        </ion-list>
      </div>

      <!-- Danger Zone -->
      <div class="settings-section">
        <ion-list-header>
          <ion-label color="danger">Danger Zone</ion-label>
        </ion-list-header>
        
        <ion-list>
          <ion-item button @click="confirmDeleteAccount" color="danger">
            <ion-icon :icon="trash" slot="start"></ion-icon>
            <ion-label>
              <h3>Delete Account</h3>
              <p>Permanently delete your account and data</p>
            </ion-label>
          </ion-item>

          <ion-item button @click="confirmSignOut" color="danger">
            <ion-icon :icon="logOut" slot="start"></ion-icon>
            <ion-label>
              <h3>Sign Out</h3>
              <p>Sign out from this device</p>
            </ion-label>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>

    <!-- Profile Modal -->
    <ion-modal :is-open="showProfileModal" @did-dismiss="showProfileModal = false">
      <ion-header>
        <ion-toolbar>
          <ion-title>Account Profile</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="showProfileModal = false">Done</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <div class="modal-profile-section">
          <div class="modal-avatar">
            <ion-icon :icon="personCircle" size="large"></ion-icon>
            <ion-button fill="clear" size="small" @click="changeAvatar">
              <ion-icon :icon="camera"></ion-icon>
            </ion-button>
          </div>
          
          <ion-item>
            <ion-label position="stacked">Full Name</ion-label>
            <ion-input v-model="editableName" placeholder="Enter your name"></ion-input>
          </ion-item>
          
          <ion-item>
            <ion-label position="stacked">Email</ion-label>
            <ion-input v-model="editableEmail" type="email" placeholder="Enter your email"></ion-input>
          </ion-item>
          
          <ion-item>
            <ion-label>Subscription Plan</ion-label>
            <ion-chip :color="subscriptionPlan === 'Premium' ? 'primary' : 'medium'">
              <ion-icon :icon="shield"></ion-icon>
              <ion-label>{{ subscriptionPlan }}</ion-label>
            </ion-chip>
          </ion-item>

          <ion-button expand="block" @click="saveProfile" class="ion-margin-top">
            Save Changes
          </ion-button>

          <ion-button expand="block" fill="outline" @click="showSubscriptionModal = true" class="ion-margin-top">
            Manage Subscription
          </ion-button>
        </div>
      </ion-content>
    </ion-modal>

    <!-- Master Password Change Modal -->
    <ion-modal :is-open="showMasterPasswordModal" @did-dismiss="showMasterPasswordModal = false">
      <ion-header>
        <ion-toolbar>
          <ion-title>Change Master Password</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="showMasterPasswordModal = false">Cancel</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <ion-item>
          <ion-label position="stacked">Current Password</ion-label>
          <ion-input type="password" v-model="currentPassword" placeholder="Enter current password"></ion-input>
        </ion-item>
        
        <ion-item>
          <ion-label position="stacked">New Password</ion-label>
          <ion-input type="password" v-model="newPassword" placeholder="Enter new password"></ion-input>
        </ion-item>
        
        <ion-item>
          <ion-label position="stacked">Confirm New Password</ion-label>
          <ion-input type="password" v-model="confirmPassword" placeholder="Confirm new password"></ion-input>
        </ion-item>

        <div class="password-strength" v-if="newPassword">
          <ion-label>Password Strength</ion-label>
          <ion-progress-bar :value="passwordStrength" :color="passwordStrengthColor"></ion-progress-bar>
          <p class="strength-text">{{ passwordStrengthText }}</p>
        </div>

        <ion-button expand="block" @click="changeMasterPassword" :disabled="!canChangePassword" class="ion-margin-top">
          Update Password
        </ion-button>
      </ion-content>
    </ion-modal>

    <!-- Import Data Modal -->
    <ion-modal :is-open="showImportModal" @did-dismiss="showImportModal = false">
      <ion-header>
        <ion-toolbar>
          <ion-title>Import Data</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="showImportModal = false">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <ion-list>
          <ion-item button>
            <ion-icon :icon="document" slot="start"></ion-icon>
            <ion-label>
              <h3>CSV File</h3>
              <p>Import from CSV file</p>
            </ion-label>
          </ion-item>
          <ion-item button>
            <ion-icon :icon="browsers" slot="start"></ion-icon>
            <ion-label>
              <h3>Browser Export</h3>
              <p>Import from browser saved passwords</p>
            </ion-label>
          </ion-item>
          <ion-item button>
            <ion-icon :icon="key" slot="start"></ion-icon>
            <ion-label>
              <h3>Other Password Managers</h3>
              <p>Import from 1Password, LastPass, etc.</p>
            </ion-label>
          </ion-item>
        </ion-list>
      </ion-content>
    </ion-modal>

    <!-- About Modal -->
    <ion-modal :is-open="showAboutModal" @did-dismiss="showAboutModal = false">
      <ion-header>
        <ion-toolbar>
          <ion-title>About</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="showAboutModal = false">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <div class="about-content">
          <div class="app-logo">
            <ion-icon :icon="shield" size="large" color="primary"></ion-icon>
            <h2>SecureVault</h2>
          </div>
          <p>Version {{ appVersion }}</p>
          <p>Build {{ buildNumber }}</p>
          <p>A secure and reliable password manager to keep your digital life safe.</p>
          
          <ion-button expand="block" fill="outline" @click="checkForUpdates">
            Check for Updates
          </ion-button>
        </div>
      </ion-content>
    </ion-modal>

    <!-- Subscription Modal -->
    <ion-modal :is-open="showSubscriptionModal" @did-dismiss="showSubscriptionModal = false">
      <ion-header>
        <ion-toolbar>
          <ion-title>Subscription</ion-title>
          <ion-buttons slot="end">
            <ion-button @click="showSubscriptionModal = false">Close</ion-button>
          </ion-buttons>
        </ion-toolbar>
      </ion-header>
      <ion-content class="ion-padding">
        <p>Subscription management will be implemented here.</p>
      </ion-content>
    </ion-modal>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { 
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem,
  IonLabel, IonToggle, IonSelect, IonSelectOption, IonButton, IonIcon,
  IonButtons, IonModal, IonInput, IonRange, IonListHeader, IonChip,
  IonProgressBar, alertController, toastController
} from '@ionic/vue';
import { 
  keyOutline, downloadOutline, personCircle, shield, chevronForward,
  fingerPrint, time, eyeOff, cloudUpload, sync, moon,
  informationCircle, help, trash, logOut, camera, document, browsers,
  key
} from 'ionicons/icons';

// User Profile Data
const userName = ref('John Doe');
const userEmail = ref('john.doe@example.com');
const subscriptionPlan = ref('Premium');
const editableName = ref(userName.value);
const editableEmail = ref(userEmail.value);

// Settings State
const darkMode = ref(false);
const autoLockTime = ref('5');
const biometricAuth = ref(false);
const hidePasswords = ref(true);
const autoSync = ref(true);

// Modal States
const showProfileModal = ref(false);
const showMasterPasswordModal = ref(false);
const showImportModal = ref(false);
const showAboutModal = ref(false);
const showSubscriptionModal = ref(false);

// Master Password Change
const currentPassword = ref('');
const newPassword = ref('');
const confirmPassword = ref('');

// App Info
const appVersion = ref('2.1.0');
const buildNumber = ref('2024.08.23');

// Computed Properties
const passwordStrength = computed(() => {
  const password = newPassword.value;
  if (!password) return 0;
  
  let score = 0;
  if (password.length >= 8) score += 0.25;
  if (password.length >= 12) score += 0.25;
  if (/[A-Z]/.test(password)) score += 0.25;
  if (/[0-9]/.test(password)) score += 0.125;
  if (/[^A-Za-z0-9]/.test(password)) score += 0.125;
  
  return Math.min(score, 1);
});

const passwordStrengthColor = computed(() => {
  const strength = passwordStrength.value;
  if (strength < 0.33) return 'danger';
  if (strength < 0.66) return 'warning';
  return 'success';
});

const passwordStrengthText = computed(() => {
  const strength = passwordStrength.value;
  if (strength < 0.33) return 'Weak';
  if (strength < 0.66) return 'Medium';
  return 'Strong';
});

const canChangePassword = computed(() => {
  return currentPassword.value && 
         newPassword.value && 
         confirmPassword.value &&
         newPassword.value === confirmPassword.value &&
         newPassword.value.length >= 8;
});

// Methods
const toggleDarkMode = () => {
  document.body.classList.toggle('dark', darkMode.value);
  showToast(`Dark mode ${darkMode.value ? 'enabled' : 'disabled'}`);
};

const toggleBiometric = () => {
  showToast(`Biometric authentication ${biometricAuth.value ? 'enabled' : 'disabled'}`);
};

const updateAutoLock = () => {
  const timeText = autoLockTime.value === '-1' ? 'never' : 
                   autoLockTime.value === '0' ? 'immediately' :
                   `${autoLockTime.value} minute${autoLockTime.value !== '1' ? 's' : ''}`;
  showToast(`Auto-lock set to ${timeText}`);
};

const toggleHidePasswords = () => {
  showToast(`Passwords will be ${hidePasswords.value ? 'hidden' : 'visible'} by default`);
};

const toggleAutoSync = () => {
  showToast(`Auto-sync ${autoSync.value ? 'enabled' : 'disabled'}`);
};

const saveProfile = () => {
  userName.value = editableName.value;
  userEmail.value = editableEmail.value;
  showProfileModal.value = false;
  showToast('Profile updated successfully', 'success');
};

const changeAvatar = () => {
  showToast('Avatar change feature coming soon', 'primary');
};

const changeMasterPassword = () => {
  if (!canChangePassword.value) return;
  
  showToast('Master password updated successfully', 'success');
  showMasterPasswordModal.value = false;
  currentPassword.value = '';
  newPassword.value = '';
  confirmPassword.value = '';
};

const exportData = () => {
  showToast('Data export started...', 'primary');
};

const contactSupport = () => {
  showToast('Opening support center...', 'primary');
};

const checkForUpdates = () => {
  showToast('Checking for updates...', 'primary');
};

const confirmDeleteAccount = async () => {
  const alert = await alertController.create({
    header: 'Delete Account',
    message: 'Are you sure you want to permanently delete your account? This action cannot be undone.',
    buttons: [
      'Cancel',
      {
        text: 'Delete',
        role: 'destructive',
        handler: () => {
          showToast('Account deletion process started', 'warning');
        }
      }
    ]
  });
  await alert.present();
};

const confirmSignOut = async () => {
  const alert = await alertController.create({
    header: 'Sign Out',
    message: 'Are you sure you want to sign out?',
    buttons: [
      'Cancel',
      {
        text: 'Sign Out',
        role: 'destructive',
        handler: () => {
          showToast('Signed out successfully');
        }
      }
    ]
  });
  await alert.present();
};

const showToast = async (message: string, color: string = 'medium') => {
  const toast = await toastController.create({
    message,
    duration: 2000,
    color,
    position: 'bottom'
  });
  await toast.present();
};
</script>

<style scoped>
.profile-card {
  display: flex;
  align-items: center;
  padding: 20px 16px;
  background: var(--ion-color-light, #f8f9fa);
  margin: 16px;
  border-radius: 12px;
  gap: 16px;
  cursor: pointer;
}

.profile-avatar {
  font-size: 60px;
  color: var(--ion-color-primary);
  cursor: pointer;
}

.profile-info {
  flex: 1;
}

.profile-info h2 {
  margin: 0 0 4px 0;
  font-size: 18px;
  font-weight: 600;
}

.profile-info p {
  margin: 0 0 8px 0;
  color: var(--ion-color-medium);
  font-size: 14px;
}

.settings-section {
  margin-bottom: 24px;
}

ion-list-header {
  padding: 16px;
  font-weight: 600;
  font-size: 16px;
  color: var(--ion-color-primary);
}

ion-list-header[color="danger"] {
  color: var(--ion-color-danger);
}

ion-list {
  padding: 0 16px;
}

ion-item {
  --padding-start: 0;
  margin-bottom: 4px;
  border-radius: 8px;
}

ion-item h3 {
  font-weight: 500;
  margin: 0;
  font-size: 16px;
}

ion-item p {
  margin: 4px 0 0 0;
  color: var(--ion-color-medium);
  font-size: 14px;
}

.modal-profile-section {
  text-align: center;
}

.modal-avatar {
  position: relative;
  display: inline-block;
  font-size: 60px;
  color: var(--ion-color-primary);
  margin-bottom: 20px;
}

.modal-avatar ion-button {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 30px;
  height: 30px;
}

.password-strength {
  margin: 16px 0;
}

.password-strength ion-label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
}

.strength-text {
  margin: 4px 0 0 0;
  font-size: 14px;
  text-align: center;
}

.about-content {
  text-align: center;
}

.app-logo {
  margin: 20px 0;
}

.app-logo ion-icon {
  font-size: 64px;
  margin-bottom: 8px;
}

.app-logo h2 {
  margin: 0;
  color: var(--ion-color-primary);
}
</style>