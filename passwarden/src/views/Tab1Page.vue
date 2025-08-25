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
      <ion-modal :is-open="isSearchOpen" @did-dismiss="closeSearch">
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
            placeholder="Search by app, website, username or email"
            @ion-input="handleSearch"
            show-clear-button="focus"
            :debounce="300"
          ></ion-searchbar>
          
          <!-- Search Results -->
          <div v-if="searchResults.length > 0" class="search-results">
            <ion-list :inset="true" class="search-list">
              <ion-item 
                v-for="result in searchResults" 
                :key="result.id" 
                button 
                lines="full"
                @click="openPasswordDetails(result)"
              >
                <ion-avatar slot="start">
                  <div class="app-icon" :style="{ backgroundColor: getIconColor(result.name) }">
                    {{ result.name.charAt(0).toUpperCase() }}
                  </div>
                </ion-avatar>
                <ion-label>
                  <h2>{{ result.name }}</h2>
                  <p>{{ result.username || result.email || 'No username' }}</p>
                  <div class="result-meta">
                    <ion-chip :color="getCategoryColor(result.category)" size="small">
                      {{ result.category }}
                    </ion-chip>
                    <span class="last-updated">{{ formatDate(result.updatedAt) }}</span>
                  </div>
                </ion-label>
                <ion-button 
                  fill="clear" 
                  @click.stop="copyPasswordQuick(result.password)"
                  class="quick-copy"
                >
                  <ion-icon :icon="copyOutline"></ion-icon>
                </ion-button>
              </ion-item>
            </ion-list>
          </div>
          
          <!-- No Search Results -->
          <div v-else-if="searchQuery && searchResults.length === 0" class="no-results">
            <ion-icon :icon="searchOutline" class="no-results-icon"></ion-icon>
            <ion-text color="medium">
              <h3>No results found</h3>
              <p>Try searching for a different app, website, username, or email</p>
            </ion-text>
          </div>

          <!-- Search Suggestions -->
          <div v-else-if="!searchQuery && savedPasswords.length > 0" class="search-suggestions">
            <h3>Recent Passwords</h3>
            <ion-list :inset="true">
              <ion-item 
                v-for="password in recentPasswords" 
                :key="password.id" 
                button 
                lines="full"
                @click="openPasswordDetails(password)"
              >
                <ion-avatar slot="start">
                  <div class="app-icon" :style="{ backgroundColor: getIconColor(password.name) }">
                    {{ password.name.charAt(0).toUpperCase() }}
                  </div>
                </ion-avatar>
                <ion-label>
                  <h2>{{ password.name }}</h2>
                  <p>{{ password.username || password.email || 'No username' }}</p>
                </ion-label>
                <ion-icon :icon="chevronForwardOutline" slot="end" color="medium"></ion-icon>
              </ion-item>
            </ion-list>
          </div>

          <!-- Empty State for Search -->
          <div v-else-if="!searchQuery && savedPasswords.length === 0" class="search-empty">
            <ion-icon :icon="lockClosedOutline" class="empty-icon"></ion-icon>
            <h3>No passwords saved yet</h3>
            <p>Add passwords to your vault to search them here</p>
          </div>
        </ion-content>
      </ion-modal>

      <!-- Password Details Modal -->
      <ion-modal :is-open="isDetailsModalOpen" @did-dismiss="closeDetailsModal">
        <ion-header>
          <ion-toolbar>
            <ion-title>{{ selectedPassword?.name }}</ion-title>
            <ion-buttons slot="end">
              <ion-button @click="closeDetailsModal" fill="clear">
                <ion-icon :icon="closeOutline"></ion-icon>
              </ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content class="ion-padding">
          <div v-if="selectedPassword" class="details-content">
            <div class="details-header">
              <div class="details-avatar">
                <div class="app-icon large" :style="{ backgroundColor: getIconColor(selectedPassword.name) }">
                  {{ selectedPassword.name.charAt(0).toUpperCase() }}
                </div>
              </div>
              <h2>{{ selectedPassword.name }}</h2>
              <ion-chip :color="getCategoryColor(selectedPassword.category)">
                {{ selectedPassword.category }}
              </ion-chip>
            </div>

            <ion-card>
              <ion-card-content>
                <ion-list lines="none">
                  <ion-item v-if="selectedPassword.username">
                    <ion-label>
                      <h3>Username</h3>
                      <p>{{ selectedPassword.username }}</p>
                    </ion-label>
                    <ion-button 
                      fill="clear" 
                      @click="copyText(selectedPassword.username)"
                      slot="end"
                    >
                      <ion-icon :icon="copyOutline"></ion-icon>
                    </ion-button>
                  </ion-item>

                  <ion-item v-if="selectedPassword.email">
                    <ion-label>
                      <h3>Email</h3>
                      <p>{{ selectedPassword.email }}</p>
                    </ion-label>
                    <ion-button 
                      fill="clear" 
                      @click="copyText(selectedPassword.email)"
                      slot="end"
                    >
                      <ion-icon :icon="copyOutline"></ion-icon>
                    </ion-button>
                  </ion-item>

                  <ion-item>
                    <ion-label>
                      <h3>Password</h3>
                      <p class="password-display">
                        {{ showDetailsPassword ? selectedPassword.password : '•'.repeat(selectedPassword.password.length) }}
                      </p>
                    </ion-label>
                    <ion-button 
                      fill="clear" 
                      @click="toggleDetailsPassword"
                      slot="end"
                    >
                      <ion-icon :icon="showDetailsPassword ? eyeOffOutline : eyeOutline"></ion-icon>
                    </ion-button>
                    <ion-button 
                      fill="clear" 
                      @click="copyText(selectedPassword.password)"
                      slot="end"
                    >
                      <ion-icon :icon="copyOutline"></ion-icon>
                    </ion-button>
                  </ion-item>

                  <ion-item v-if="selectedPassword.notes">
                    <ion-label>
                      <h3>Notes</h3>
                      <p>{{ selectedPassword.notes }}</p>
                    </ion-label>
                  </ion-item>
                </ion-list>
              </ion-card-content>
            </ion-card>

            <div class="details-meta">
              <p>Created: {{ formatDate(selectedPassword.createdAt) }}</p>
              <p>Last updated: {{ formatDate(selectedPassword.updatedAt) }}</p>
            </div>
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
              <div class="stat-item secondary">
                <ion-icon :icon="trendingUpOutline" class="stat-icon"></ion-icon>
                <div class="stat-info">
                  <h2>{{ weeklyAdditions }}</h2>
                  <p>Added This Week</p>
                </div>
              </div>
            </div>
          </ion-card-content>
        </ion-card>

        <!-- Quick Actions -->
        <div class="section-header">
          <ion-text>
            <h2>Quick Actions</h2>
          </ion-text>
        </div>

        <div class="actions-grid">
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

          <ion-card button @click="openSearch" class="action-card search-card">
            <ion-card-content>
              <div class="action-content">
                <div class="action-icon-container search">
                  <ion-icon :icon="searchOutline" class="action-icon"></ion-icon>
                </div>
                <div class="action-text">
                  <h3>Quick Search</h3>
                  <p>Find your passwords instantly</p>
                </div>
                <ion-icon :icon="chevronForwardOutline" class="action-chevron"></ion-icon>
              </div>
            </ion-card-content>
          </ion-card>
        </div>

        <!-- Recent Activity Card -->
        <ion-card v-if="recentActivity.length > 0" class="recent-card">
          <ion-card-header>
            <ion-card-title>Recent Activity</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <div 
              v-for="activity in recentActivity" 
              :key="activity.id"
              class="activity-item"
            >
              <div class="activity-icon">
                <ion-icon :icon="activity.icon" :color="activity.color"></ion-icon>
              </div>
              <div class="activity-text">
                <p class="activity-title">{{ activity.title }}</p>
                <p class="activity-subtitle">{{ activity.subtitle }}</p>
              </div>
            </div>
          </ion-card-content>
        </ion-card>

        <!-- Password Generator Modal -->
        <ion-modal :is-open="isGeneratorOpen" @did-dismiss="closeGenerator">
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
import { ref, onMounted, computed } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons, IonButton,
  IonIcon, IonModal, IonSearchbar, IonList, IonItem, IonLabel, IonAvatar,
  IonText, IonCard, IonCardContent, IonCardHeader, IonCardTitle, IonTextarea,
  IonRange, IonCheckbox, IonChip
} from '@ionic/vue';
import {
  searchOutline, keyOutline, copyOutline, refreshOutline, closeOutline,
  chevronForwardOutline, shieldCheckmarkOutline, lockClosedOutline,
  trendingUpOutline, addCircleOutline, refreshCircleOutline,
  eyeOutline, eyeOffOutline
} from 'ionicons/icons';
import { Clipboard } from '@capacitor/clipboard';
import { Haptics, ImpactStyle } from '@capacitor/haptics';

// Interfaces
interface Password {
  id: string;
  name: string;
  username?: string;
  email?: string;
  password: string;
  category: string;
  notes?: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Activity {
  id: string;
  title: string;
  subtitle: string;
  icon: string;
  color: string;
}

// State
const savedPasswords = ref<Password[]>([]);
const totalPasswords = computed(() => savedPasswords.value.length);
const weeklyAdditions = computed(() => {
  const weekAgo = new Date();
  weekAgo.setDate(weekAgo.getDate() - 7);
  return savedPasswords.value.filter(p => new Date(p.createdAt) > weekAgo).length;
});

const isSearchOpen = ref(false);
const isGeneratorOpen = ref(false);
const isDetailsModalOpen = ref(false);
const searchQuery = ref('');
const searchResults = ref<Password[]>([]);
const selectedPassword = ref<Password | null>(null);
const showDetailsPassword = ref(false);

// Password Generator State
const generatedPassword = ref('');
const passwordLength = ref(16);
const includeUppercase = ref(true);
const includeLowercase = ref(true);
const includeNumbers = ref(true);
const includeSymbols = ref(true);

// Color palette for app icons
const iconColors = [
  '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7',
  '#DDA0DD', '#98D8C8', '#F7DC6F', '#BB8FCE', '#85C1E9'
];

const categoryColors: Record<string, string> = {
  social: 'primary',
  work: 'secondary',
  banking: 'success',
  shopping: 'warning',
  entertainment: 'tertiary',
  other: 'medium'
};

// Computed properties
const recentPasswords = computed(() => {
  return savedPasswords.value
    .sort((a, b) => new Date(b.updatedAt).getTime() - new Date(a.updatedAt).getTime())
    .slice(0, 5);
});

const recentActivity = computed(() => {
  const activities: Activity[] = [];
  
  // Get recent passwords (added in last 7 days)
  const weekAgo = new Date();
  weekAgo.setDate(weekAgo.getDate() - 7);
  
  const recentlyAdded = savedPasswords.value
    .filter(p => new Date(p.createdAt) > weekAgo)
    .sort((a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime())
    .slice(0, 2);

  // Get recently updated passwords
  const recentlyUpdated = savedPasswords.value
    .filter(p => {
      const created = new Date(p.createdAt);
      const updated = new Date(p.updatedAt);
      return updated > weekAgo && updated.getTime() !== created.getTime();
    })
    .sort((a, b) => new Date(b.updatedAt).getTime() - new Date(a.updatedAt).getTime())
    .slice(0, 2);

  // Add activities
  recentlyAdded.forEach(password => {
    activities.push({
      id: `added-${password.id}`,
      title: 'Password added',
      subtitle: `${password.name} • ${getRelativeTime(password.createdAt)}`,
      icon: addCircleOutline,
      color: 'success'
    });
  });

  recentlyUpdated.forEach(password => {
    activities.push({
      id: `updated-${password.id}`,
      title: 'Password updated',
      subtitle: `${password.name} • ${getRelativeTime(password.updatedAt)}`,
      icon: refreshCircleOutline,
      color: 'primary'
    });
  });

  return activities.slice(0, 3);
});

// Methods
const loadFromStorage = () => {
  try {
    const stored = localStorage.getItem('vault-passwords');
    if (stored) {
      const parsed = JSON.parse(stored);
      savedPasswords.value = parsed.map((p: any) => ({
        ...p,
        createdAt: new Date(p.createdAt),
        updatedAt: new Date(p.updatedAt)
      }));
    }
  } catch (error) {
    console.error('Failed to load from storage:', error);
  }
};

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
  searchQuery.value = query;
  
  if (!query) {
    searchResults.value = [];
    return;
  }
  
  searchResults.value = savedPasswords.value.filter(password =>
    password.name.toLowerCase().includes(query) ||
    (password.username?.toLowerCase() || '').includes(query) ||
    (password.email?.toLowerCase() || '').includes(query) ||
    password.category.toLowerCase().includes(query) ||
    (password.notes?.toLowerCase() || '').includes(query)
  );
};

const openPasswordDetails = (password: Password) => {
  selectedPassword.value = password;
  showDetailsPassword.value = false;
  isDetailsModalOpen.value = true;
  closeSearch();
};

const closeDetailsModal = () => {
  isDetailsModalOpen.value = false;
  selectedPassword.value = null;
};

const toggleDetailsPassword = () => {
  showDetailsPassword.value = !showDetailsPassword.value;
};

const copyPasswordQuick = async (password: string) => {
  await copyText(password);
  // Add haptic feedback
  try {
    await Haptics.impact({ style: ImpactStyle.Light });
  } catch (error) {
    // Haptics not supported, ignore
  }
};

const getIconColor = (name: string) => {
  const index = name.charCodeAt(0) % iconColors.length;
  return iconColors[index];
};

const getCategoryColor = (category: string) => {
  return categoryColors[category] || 'medium';
};

const formatDate = (date: Date) => {
  return new Date(date).toLocaleDateString();
};

const getRelativeTime = (date: Date) => {
  const now = new Date();
  const diffTime = Math.abs(now.getTime() - new Date(date).getTime());
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));
  const diffHours = Math.floor(diffTime / (1000 * 60 * 60));
  
  if (diffDays === 0) {
    if (diffHours === 0) {
      return 'Just now';
    } else if (diffHours === 1) {
      return '1 hour ago';
    } else {
      return `${diffHours} hours ago`;
    }
  } else if (diffDays === 1) {
    return 'Yesterday';
  } else if (diffDays < 7) {
    return `${diffDays} days ago`;
  } else {
    return formatDate(date);
  }
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
  await copyText(generatedPassword.value);
};

const copyText = async (text: string) => {
  try {
    await Clipboard.write({
      string: text
    });
    console.log('Copied to clipboard');
  } catch (error) {
    // Fallback for web/platforms that don't support Capacitor
    try {
      await navigator.clipboard.writeText(text);
      console.log('Copied to clipboard (fallback)');
    } catch (fallbackError) {
      console.error('Failed to copy:', fallbackError);
    }
  }
};

onMounted(() => {
  loadFromStorage();
});
</script>

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

/* Actions Grid */
.actions-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
  margin-bottom: 32px;
}

/* Action Cards */
.action-card {
  border-radius: 16px;
  transition: all 0.3s ease;
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

.action-icon-container.search {
  background: var(--ion-color-secondary);
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

.app-icon.large {
  width: 80px;
  height: 80px;
  font-size: 32px;
  border-radius: 20px;
}

.result-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 4px;
}

.last-updated {
  font-size: 12px;
  color: var(--ion-color-medium);
}

.quick-copy {
  --color: var(--ion-color-primary);
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

/* Search Suggestions */
.search-suggestions {
  padding: 16px 0;
}

.search-suggestions h3 {
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 16px 16px;
  color: var(--ion-text-color);
}

.search-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 40px;
  text-align: center;
  min-height: 40vh;
}

.empty-icon {
  font-size: 80px;
  color: var(--ion-color-medium);
  margin-bottom: 24px;
}

.search-empty h3 {
  font-size: 20px;
  font-weight: 600;
  color: var(--ion-text-color);
  margin: 0 0 8px 0;
}

.search-empty p {
  font-size: 16px;
  color: var(--ion-color-medium);
  margin: 0;
}

/* Details Modal */
.details-content {
  padding-bottom: 40px;
}

.details-header {
  text-align: center;
  margin-bottom: 24px;
}

.details-avatar {
  margin-bottom: 16px;
}

.details-header h2 {
  font-size: 28px;
  font-weight: 700;
  margin: 0 0 12px 0;
  color: var(--ion-text-color);
}

.password-display {
  font-family: 'SF Mono', 'Monaco', 'Inconsolata', 'Roboto Mono', monospace;
  font-size: 16px;
  word-break: break-all;
}

.details-meta {
  margin-top: 24px;
  text-align: center;
}

.details-meta p {
  font-size: 14px;
  color: var(--ion-color-medium);
  margin: 4px 0;
}

/* Generator Modal */
.generator-content {
  padding-bottom: 100px;
}

.password-display-card {
  margin-bottom: 20px;
  border-radius: 16px;
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
  border-radius: 16px;
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

/* Custom range styling */
.custom-length-range {
  --bar-background: #e0e0e0;
  --bar-background-active: var(--ion-color-primary);
  --bar-height: 6px;
  --knob-background: var(--ion-color-primary);
  --knob-size: 20px;
  --pin-background: var(--ion-color-primary);
  --pin-color: white;
}
</style>