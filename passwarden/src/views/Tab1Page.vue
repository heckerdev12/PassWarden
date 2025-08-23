<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Home</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="openSearch" fill="clear" shape="round">
            <ion-icon :icon="searchOutline" size="large"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true" class="ion-padding">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Home</ion-title>
        </ion-toolbar>
      </ion-header>

      <!-- Search Modal -->
      <ion-modal :is-open="isSearchOpen" @did-dismiss="closeSearch" presenting-element="$parent.$refs.page">
        <ion-header :translucent="true">
          <ion-toolbar>
            <ion-title>Search Passwords</ion-title>
            <ion-buttons slot="end">
              <ion-button @click="closeSearch" fill="clear">
                <ion-icon :icon="closeOutline"></ion-icon>
              </ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content :fullscreen="true" class="ion-padding">
          <ion-searchbar 
            v-model="searchQuery" 
            placeholder="Search by app or website name"
            @ion-input="handleSearch"
            show-clear-button="focus"
            :debounce="300"
          ></ion-searchbar>
          
          <div v-if="searchResults.length > 0" class="search-results">
            <ion-list :inset="true" class="search-list">
              <ion-item v-for="result in searchResults" key="result.id" button lines="full">
                <ion-avatar slot="start">
                  <div class="app-icon" :style="{ backgroundColor: getIconColor(result.name) }">
                    {{ result.name.charAt(0).toUpperCase() }}
                  </div>
                </ion-avatar>
                <ion-label>
                  <h2>{{ result?.name || 'Unnamed' }}</h2>
                  <p>{{ result?.username || 'No username' }}</p>
                </ion-label>
                <ion-icon :icon="chevronForwardOutline" slot="end" color="medium"></ion-icon>
              </ion-item>
            </ion-list>
          </div>
          
          <div v-else-if="searchQuery && searchResults.length === 0" class="no-results">
            <ion-icon :icon="searchOutline" class="no-results-icon"></ion-icon>
            <ion-text color="medium">
              <h3>No results found</h3>
              <p>Try searching for a different app or website name</p>
            </ion-text>
          </div>
        </ion-content>
      </ion-modal>

      <div class="home-content">
        <!-- Welcome Card -->
        <ion-card class="welcome-card">
          <ion-card-content>
            <div class="welcome-content">
              <div class="welcome-text">
                <h1>Welcome back, Jash!</h1>
                <p>Your digital life, secured and protected.</p>
              </div>
              <div class="welcome-icon">
                <ion-icon :icon="shieldCheckmarkOutline"></ion-icon>
              </div>
            </div>
          </ion-card-content>
        </ion-card>

        <!-- Stats Overview -->
        <ion-card class="stats-card">
          <ion-card-content>
            <div class="stats-grid">
              <div class="stat-item primary">
                <ion-icon :icon="lockClosedOutline" class="stat-icon"></ion-icon>
                <div class="stat-info">
                  <h2>{{ totalPasswords }}</h2>
                  <p>Total Passwords</p>
                </div>
              </div>
              <div class="stat-divider"></div>
            </div>
          </ion-card-content>
        </ion-card>

        <!-- Quick Actions -->
        <div class="section-header">
          <ion-text>
            <h2>Quick Actions</h2>
          </ion-text>
        </div>

        <ion-card button @click="generatePassword" class="action-card generator-card">
          <ion-card-content>
            <div class="action-content">
              <div class="action-icon-container">
                <ion-icon :icon="keyOutline" class="action-icon"></ion-icon>
              </div>
              <div class="action-text">
                <h3>Password Generator</h3>
                <p>Create a secure, randomized password</p>
              </div>
              <ion-icon :icon="chevronForwardOutline" class="action-chevron"></ion-icon>
            </div>
          </ion-card-content>
        </ion-card>

        <!-- Recent Activity Card -->
        <ion-card class="recent-card">
          <ion-card-header>
            <ion-card-title>Recent Activity</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <div class="activity-item">
              <div class="activity-icon">
                <ion-icon :icon="addCircleOutline" color="success"></ion-icon>
              </div>
              <div class="activity-text">
                <p class="activity-title">Password added</p>
                <p class="activity-subtitle">Instagram • 2 hours ago</p>
              </div>
            </div>
            <div class="activity-item">
              <div class="activity-icon">
                <ion-icon :icon="refreshCircleOutline" color="primary"></ion-icon>
              </div>
              <div class="activity-text">
                <p class="activity-title">Password updated</p>
                <p class="activity-subtitle">Gmail • Yesterday</p>
              </div>
            </div>
          </ion-card-content>
        </ion-card>

        <!-- Password Generator Modal -->
        <ion-modal :is-open="isGeneratorOpen" @did-dismiss="closeGenerator" presenting-element="$parent.$refs.page">
          <ion-header :translucent="true">
            <ion-toolbar>
              <ion-title>Password Generator</ion-title>
              <ion-buttons slot="end">
                <ion-button @click="closeGenerator" fill="clear">
                  <ion-icon :icon="closeOutline"></ion-icon>
                </ion-button>
              </ion-buttons>
            </ion-toolbar>
          </ion-header>
          <ion-content :fullscreen="true" class="ion-padding">
            <div class="generator-content">
              <!-- Generated Password Display -->
              <ion-card class="password-display-card">
                <ion-card-content>
                  <div class="password-strength">
                    <ion-text color="success">
                      <h3>Strong Password</h3>
                    </ion-text>
                    <div class="strength-bar">
                      <div class="strength-fill strong"></div>
                    </div>
                  </div>
                  <div class="generated-password">
                    <ion-textarea 
                      :value="generatedPassword" 
                      :readonly="true"
                      :auto-grow="true"
                      class="password-textarea"
                    ></ion-textarea>
                    <ion-button 
                      v-if="generatedPassword" 
                      fill="clear" 
                      @click="copyPassword"
                      class="copy-button"
                    >
                      <ion-icon :icon="copyOutline"></ion-icon>
                    </ion-button>
                  </div>
                </ion-card-content>
              </ion-card>

              <!-- Generator Options -->
              <ion-card class="options-card">
                <ion-card-header>
                  <ion-card-title>Customize Password</ion-card-title>
                </ion-card-header>
                <ion-card-content>
                  <ion-list lines="none" class="password-length-list">
                    <ion-item class="password-length-item">
                      <ion-range 
                        v-model="passwordLength" 
                        label="password Length"
                        :min="8" 
                        :max="50" 
                        :pin="true"
                        @ion-change="generateNewPassword"
                        class="custom-length-range"
                        :ticks="true"
                        :snaps="true"
                        :step="1"
                      >
                      </ion-range>
                    </ion-item>

                    <ion-item>
                      <ion-checkbox 
                        v-model="includeUppercase" 
                        @ion-change="generateNewPassword"
                        slot="start"
                      ></ion-checkbox>
                      <ion-label class="option-label">
                        <h3>Uppercase Letters</h3>
                        <p>A, B, C, D...</p>
                      </ion-label>
                    </ion-item>

                    <ion-item>
                      <ion-checkbox 
                        v-model="includeLowercase" 
                        @ion-change="generateNewPassword"
                        slot="start"
                      ></ion-checkbox>
                      <ion-label class="option-label">
                        <h3>Lowercase Letters</h3>
                        <p>a, b, c, d...</p>
                      </ion-label>
                    </ion-item>

                    <ion-item>
                      <ion-checkbox 
                        v-model="includeNumbers" 
                        @ion-change="generateNewPassword"
                        slot="start"
                      ></ion-checkbox>
                      <ion-label class="option-label">
                        <h3>Numbers</h3>
                        <p>1, 2, 3, 4...</p>
                      </ion-label>
                    </ion-item>

                    <ion-item>
                      <ion-checkbox 
                        v-model="includeSymbols" 
                        @ion-change="generateNewPassword"
                        slot="start"
                      ></ion-checkbox>
                      <ion-label class="option-label">
                        <h3>Symbols</h3>
                        <p>!, @, #, $...</p>
                      </ion-label>
                    </ion-item>
                  </ion-list>
                </ion-card-content>
              </ion-card>

              <div class="generator-actions">
                <ion-button expand="block" @click="generateNewPassword" size="large" color="light">
                  <ion-icon :icon="refreshOutline" slot="start"></ion-icon>
                  Generate New Password
                </ion-button>
              </div>
            </div>
          </ion-content>
        </ion-modal>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons, IonButton,
  IonIcon, IonModal, IonSearchbar, IonList, IonItem, IonLabel, IonAvatar,
  IonText, IonCard, IonCardContent, IonCardHeader, IonCardTitle, IonTextarea,
  IonRange, IonCheckbox
} from '@ionic/vue';
import {
  searchOutline, keyOutline, copyOutline, refreshOutline, closeOutline,
  chevronForwardOutline, shieldCheckmarkOutline, lockClosedOutline,
  trendingUpOutline, addCircleOutline, refreshCircleOutline
} from 'ionicons/icons';
import { Clipboard } from '@capacitor/clipboard';
import { Haptics, ImpactStyle } from '@capacitor/haptics';

// State
const totalPasswords = ref(123);
const isSearchOpen = ref(false);
const isGeneratorOpen = ref(false);
const searchQuery = ref('');
const searchResults = ref([]);

// Password Generator State
const generatedPassword = ref('');
const passwordLength = ref(16);
const includeUppercase = ref(true);
const includeLowercase = ref(true);
const includeNumbers = ref(true);
const includeSymbols = ref(true);

// Mock data for search
const mockPasswords = [
  { id: 1, name: 'Facebook', username: 'jash@email.com' },
  { id: 2, name: 'Gmail', username: 'jash@gmail.com' },
  { id: 3, name: 'Instagram', username: '@jash_dev' },
  { id: 4, name: 'Twitter', username: '@jash' },
  { id: 5, name: 'LinkedIn', username: 'jash-dev' },
];

// Color palette for app icons
const iconColors = [
  '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7',
  '#DDA0DD', '#98D8C8', '#F7DC6F', '#BB8FCE', '#85C1E9'
];

// Methods
const openSearch = () => {
  isSearchOpen.value = true;
};

const closeSearch = () => {
  isSearchOpen.value = false;
  searchQuery.value = '';
  searchResults.value = [];
};

const handleSearch = (event: CustomEvent) => {
  const query = (event.detail.value?.toLowerCase() || '').trim();
  if (!query) {
    searchResults.value = [];
    return;
  }
  
  searchResults.value = mockPasswords.filter(password =>
    (password.name?.toLowerCase() || '').includes(query) ||
    (password.username?.toLowerCase() || '').includes(query)
  );
};

const getIconColor = (name: string) => {
  const index = name.charCodeAt(0) % iconColors.length;
  return iconColors[index];
};

const generatePassword = () => {
  isGeneratorOpen.value = true;
  generateNewPassword();
};

const closeGenerator = () => {
  isGeneratorOpen.value = false;
};

const generateNewPassword = () => {
  let charset = '';
  
  if (includeUppercase.value) charset += 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  if (includeLowercase.value) charset += 'abcdefghijklmnopqrstuvwxyz';
  if (includeNumbers.value) charset += '0123456789';
  if (includeSymbols.value) charset += '!@#$%^&*()_+-=[]{}|;:,.<>?';
  
  if (charset === '') {
    generatedPassword.value = '';
    return;
  }
  
  let password = '';
  for (let i = 0; i < passwordLength.value; i++) {
    password += charset.charAt(Math.floor(Math.random() * charset.length));
  }
  
  generatedPassword.value = password;
};

const copyPassword = async () => {
  try {
    await Clipboard.write({
      string: generatedPassword.value
    });
    
    await Haptics.impact({ style: ImpactStyle.Light });
    console.log('Password copied to clipboard');
  } catch (error) {
    console.error('Failed to copy password:', error);
  }
};

onMounted(() => {
  // Initialize any data loading here
});
</script>

<style scoped>
/* Custom range styling without colors */
.custom-length-range {
  --bar-background: #e0e0e0;
  --bar-background-active: var(--ion-color-primary);
  --bar-height: 6px;
  --knob-background: var(--ion-color-primary);
  --knob-size: 20px;
  --pin-background: var(--ion-color-primary);
  --pin-color: white;
}

.custom-length-range::part(bar) {
  background: var(--bar-background);
  height: var(--bar-height);
  border-radius: 3px;
}

.custom-length-range::part(bar-active) {
  background: var(--bar-background-active);
}

.custom-length-range::part(knob) {
  background: var(--knob-background);
  width: var(--knob-size);
  height: var(--knob-size);
  border: 2px solid white;
  box-shadow: 0 2px 6px rgba(0,0,0,0.15);
}

.custom-length-range::part(pin) {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  background: var(--pin-background);
  color: var(--pin-color);
  border-radius: 50%;
  transform: scale(1);
  top: -22px;
  min-width: 28px;
  height: 28px;
  font-weight: 600;
  font-size: 12px;
  transition: transform 120ms ease, background 120ms ease;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}

.custom-length-range::part(pin)::before {
  content: none;
}

/* Dark mode adjustments */
@media (prefers-color-scheme: dark) {
  .custom-length-range {
    --bar-background: var(--ion-color-step-200);
  }
}
</style>
<style scoped>
.home-content {
  padding-bottom: 100px;
}

/* Welcome Card */
.welcome-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  margin-bottom: 20px;
  border-radius: 16px;
  overflow: hidden;
}

.welcome-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 0;
}

.welcome-text h1 {
  color: white;
  font-size: 28px;
  font-weight: 700;
  margin: 0 0 8px 0;
  line-height: 1.2;
}

.welcome-text p {
  color: rgba(255, 255, 255, 0.8);
  font-size: 16px;
  margin: 0;
  line-height: 1.4;
}

.welcome-icon {
  opacity: 0.2;
}

.welcome-icon ion-icon {
  font-size: 80px;
  color: white;
}

/* Stats Card */
.stats-card {
  margin-bottom: 24px;
  border-radius: 16px;
  background: var(--ion-color-step-50);
}

.stats-grid {
  display: flex;
  align-items: center;
  gap: 20px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 12px;
  flex: 1;
}

.stat-icon {
  font-size: 24px;
  color: var(--ion-color-primary);
}

.stat-info h2 {
  font-size: 32px;
  font-weight: 700;
  margin: 0;
  color: var(--ion-color-primary);
}

.stat-info p {
  font-size: 14px;
  color: var(--ion-color-medium);
  margin: 0;
}

.stat-divider {
  width: 1px;
  height: 40px;
  background: var(--ion-color-step-150);
}

/* Section Headers */
.section-header {
  margin: 32px 0 16px 0;
}

.section-header h2 {
  font-size: 24px;
  font-weight: 700;
  color: var(--ion-text-color);
  margin: 0;
}

/* Action Cards */
.action-card {
  margin-bottom: 16px;
  border-radius: 16px;
  transition: all 0.3s ease;
  background: var(--ion-color-step-50);
}

.action-card:hover {
  transform: translateY(-2px);
}

.action-content {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 4px 0;
}

.action-icon-container {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  background: var(--ion-color-primary);
  display: flex;
  align-items: center;
  justify-content: center;
}

.action-icon {
  font-size: 24px;
  color: white;
}

.action-text {
  flex: 1;
}

.action-text h3 {
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 4px 0;
  color: var(--ion-text-color);
}

.action-text p {
  font-size: 14px;
  color: var(--ion-color-medium);
  margin: 0;
}

.action-chevron {
  font-size: 20px;
  color: var(--ion-color-medium);
}

/* Recent Activity Card */
.recent-card {
  border-radius: 16px;
  background: var(--ion-color-step-50);
}

.activity-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 0;
}

.activity-item:not(:last-child) {
  border-bottom: 1px solid var(--ion-color-step-100);
}

.activity-icon {
  display: flex;
  align-items: center;
  justify-content: center;
}

.activity-icon ion-icon {
  font-size: 24px;
}

.activity-text {
  flex: 1;
}

.activity-title {
  font-size: 16px;
  font-weight: 500;
  margin: 0 0 2px 0;
  color: var(--ion-text-color);
}

.activity-subtitle {
  font-size: 14px;
  color: var(--ion-color-medium);
  margin: 0;
}

/* Search Results */
.search-list {
  margin-top: 16px;
}

.search-results {
  margin-top: 16px;
}

.app-icon {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 18px;
}

.no-results {
  padding: 60px 16px;
  text-align: center;
}

.no-results-icon {
  font-size: 64px;
  color: var(--ion-color-medium);
  margin-bottom: 16px;
}

.no-results h3 {
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 8px 0;
}

.no-results p {
  font-size: 14px;
  margin: 0;
}

/* Generator Modal */
.generator-content {
  padding-bottom: 100px;
}

.password-display-card {
  margin-bottom: 20px;
  border-radius: 16px;
  background: var(--ion-color-step-50);
}

.password-strength {
  margin-bottom: 16px;
}

.password-strength h3 {
  font-size: 16px;
  font-weight: 600;
  margin: 0 0 8px 0;
}

.strength-bar {
  height: 4px;
  background: var(--ion-color-step-150);
  border-radius: 2px;
  overflow: hidden;
}

.strength-fill {
  height: 100%;
  border-radius: 2px;
  transition: all 0.3s ease;
}

.strength-fill.strong {
  width: 100%;
  background: var(--ion-color-success);
}

.generated-password {
  display: flex;
  align-items: flex-start;
  gap: 8px;
}

.password-textarea {
  flex: 1;
  font-family: 'SF Mono', 'Monaco', 'Inconsolata', 'Roboto Mono', monospace;
  font-size: 16px;
  line-height: 1.4;
  --padding-start: 0;
  --padding-end: 0;
}

.copy-button {
  margin-top: 8px;
}

.options-card {
  margin-bottom: 20px;
  margin-left: 5px;
  margin-right: 5px;
  border-radius: 16px;
  background: var(--ion-color-step-50);
}
.options-content {
  padding: 40px;
}

.option-label h3 {
  font-size: 16px;
  font-weight: 500;
  margin: 0 0 2px 0;
}

.option-label p {
  font-size: 14px;
  color: var(--ion-color-medium);
  margin: 0;
}

.generator-actions {
  padding: 24px 0;
}

/* iOS-style improvements */
ion-card {
  --background: var(--ion-color-step-50);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

ion-item {
  --background: transparent;
  --inner-padding-end: 0;
  --padding-start: 15px;
}

ion-button {
  --border-radius: 12px;
}

ion-modal {
  --border-radius: 16px;
}

/* Dark theme specific adjustments */
@media (prefers-color-scheme: dark) {
  .welcome-card {
    background: linear-gradient(135deg, #4a5568 0%, #2d3748 100%);
  }
  
  ion-card {
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  }
}
</style>