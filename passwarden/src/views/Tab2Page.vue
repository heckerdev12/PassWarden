<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Vault</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="openSearch" fill="clear" shape="round">
            <ion-icon :icon="searchOutline" size="large"></ion-icon>
          </ion-button>
          <ion-button @click="openAddModal" fill="clear" shape="round">
            <ion-icon :icon="addOutline" size="large"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Vault</ion-title>
        </ion-toolbar>
      </ion-header>

      <!-- Search Bar (toggleable) -->
      <div v-if="showSearch" class="search-container">
        <ion-searchbar 
          v-model="searchQuery" 
          placeholder="Search passwords..."
          @ion-input="handleSearch"
          show-clear-button="focus"
          :debounce="300"
        ></ion-searchbar>
      </div>

      <!-- Empty State -->
      <div v-if="filteredPasswords.length === 0 && !searchQuery" class="empty-state">
        <ion-icon :icon="lockClosedOutline" class="empty-icon"></ion-icon>
        <h2>Your vault is empty</h2>
        <p>Start securing your passwords by adding your first entry</p>
        <ion-button @click="openAddModal" expand="block" class="add-first-button">
          <ion-icon :icon="addOutline" slot="start"></ion-icon>
          Add Your First Password
        </ion-button>
      </div>

      <!-- No Search Results -->
      <div v-else-if="filteredPasswords.length === 0 && searchQuery" class="no-results">
        <ion-icon :icon="searchOutline" class="no-results-icon"></ion-icon>
        <h3>No results found</h3>
        <p>Try searching with different keywords</p>
      </div>

      <!-- Password List -->
      <div v-else class="vault-content">
        <!-- Categories -->
        <div v-if="!searchQuery" class="categories">
          <ion-segment v-model="selectedCategory" @ion-change="filterByCategory">
            <ion-segment-button value="all">
              <ion-label>All</ion-label>
            </ion-segment-button>
            <ion-segment-button value="social">
              <ion-label>Social</ion-label>
            </ion-segment-button>
            <ion-segment-button value="work">
              <ion-label>Work</ion-label>
            </ion-segment-button>
            <ion-segment-button value="banking">
              <ion-label>Banking</ion-label>
            </ion-segment-button>
            <ion-segment-button value="other">
              <ion-label>Other</ion-label>
            </ion-segment-button>
          </ion-segment>
        </div>

        <!-- Password Cards -->
        <div class="passwords-container">
          <ion-card 
            v-for="password in filteredPasswords" 
            :key="password.id"
            button
            @click="openViewModal(password)"
            class="password-card"
          >
            <ion-card-content>
              <div class="password-item">
                <div class="password-avatar">
                  <div class="app-icon" :style="{ backgroundColor: getIconColor(password.name) }">
                    {{ password.name.charAt(0).toUpperCase() }}
                  </div>
                </div>
                <div class="password-info">
                  <h3>{{ password.name }}</h3>
                  <p>{{ password.username || password.email || 'No username' }}</p>
                  <div class="password-meta">
                    <ion-chip :color="getCategoryColor(password.category)" size="small">
                      {{ password.category }}
                    </ion-chip>
                    <span class="last-updated">{{ formatDate(password.updatedAt) }}</span>
                  </div>
                </div>
                <div class="password-actions">
                  <ion-button 
                    fill="clear" 
                    @click.stop="copyPassword(password.password)"
                    class="copy-btn"
                  >
                    <ion-icon :icon="copyOutline"></ion-icon>
                  </ion-button>
                  <ion-icon :icon="chevronForwardOutline" color="medium"></ion-icon>
                </div>
              </div>
            </ion-card-content>
          </ion-card>
        </div>
      </div>

      <!-- Add Password Modal -->
      <ion-modal :is-open="isAddModalOpen" @did-dismiss="closeAddModal">
        <ion-header>
          <ion-toolbar>
            <ion-title>{{ editingPassword ? 'Edit Password' : 'Add Password' }}</ion-title>
            <ion-buttons slot="end">
              <ion-button @click="closeAddModal" fill="clear">
                <ion-icon :icon="closeOutline"></ion-icon>
              </ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content class="ion-padding">
          <form @submit.prevent="savePassword">
            <ion-item>
              <ion-input
                v-model="newPassword.name"
                label="App/Website Name"
                label-placement="stacked"
                placeholder="e.g. Gmail, Facebook"
                required
              ></ion-input>
            </ion-item>

            <ion-item>
              <ion-input
                v-model="newPassword.username"
                label="Username"
                label-placement="stacked"
                placeholder="Username or email"
              ></ion-input>
            </ion-item>

            <ion-item>
              <ion-input
                v-model="newPassword.email"
                label="Email"
                label-placement="stacked"
                placeholder="Email address"
                type="email"
              ></ion-input>
            </ion-item>

            <ion-item>
              <ion-input
                v-model="newPassword.password"
                :type="showPassword ? 'text' : 'password'"
                label="Password"
                label-placement="stacked"
                placeholder="Enter password"
                required
              ></ion-input>
              <ion-button 
                @click="togglePasswordVisibility" 
                fill="clear" 
                slot="end"
              >
                <ion-icon :icon="showPassword ? eyeOffOutline : eyeOutline"></ion-icon>
              </ion-button>
            </ion-item>

            <ion-item>
              <ion-select
                v-model="newPassword.category"
                label="Category"
                label-placement="stacked"
                placeholder="Select category"
              >
                <ion-select-option value="social">Social</ion-select-option>
                <ion-select-option value="work">Work</ion-select-option>
                <ion-select-option value="banking">Banking</ion-select-option>
                <ion-select-option value="shopping">Shopping</ion-select-option>
                <ion-select-option value="entertainment">Entertainment</ion-select-option>
                <ion-select-option value="other">Other</ion-select-option>
              </ion-select>
            </ion-item>

            <ion-item>
              <ion-textarea
                v-model="newPassword.notes"
                label="Notes"
                label-placement="stacked"
                placeholder="Additional notes (optional)"
                :auto-grow="true"
              ></ion-textarea>
            </ion-item>

            <ion-button 
              type="submit" 
              expand="block" 
              class="save-button"
            >
              {{ editingPassword ? 'Update Password' : 'Save Password' }}
            </ion-button>

            <ion-button 
              v-if="editingPassword"
              @click="deletePassword"
              expand="block" 
              fill="outline"
              color="danger"
              class="delete-button"
            >
              Delete Password
            </ion-button>
          </form>
        </ion-content>
      </ion-modal>

      <!-- View Password Modal -->
      <ion-modal :is-open="isViewModalOpen" @did-dismiss="closeViewModal">
        <ion-header>
          <ion-toolbar>
            <ion-title>{{ viewingPassword?.name }}</ion-title>
            <ion-buttons slot="end">
              <ion-button @click="editPassword(viewingPassword)" fill="clear">
                <ion-icon :icon="pencilOutline"></ion-icon>
              </ion-button>
              <ion-button @click="closeViewModal" fill="clear">
                <ion-icon :icon="closeOutline"></ion-icon>
              </ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content class="ion-padding">
          <div v-if="viewingPassword" class="view-content">
            <div class="view-header">
              <div class="view-avatar">
                <div class="app-icon large" :style="{ backgroundColor: getIconColor(viewingPassword.name) }">
                  {{ viewingPassword.name.charAt(0).toUpperCase() }}
                </div>
              </div>
              <h2>{{ viewingPassword.name }}</h2>
              <ion-chip :color="getCategoryColor(viewingPassword.category)">
                {{ viewingPassword.category }}
              </ion-chip>
            </div>

            <ion-card>
              <ion-card-content>
                <ion-list lines="none">
                  <ion-item v-if="viewingPassword.username">
                    <ion-label>
                      <h3>Username</h3>
                      <p>{{ viewingPassword.username }}</p>
                    </ion-label>
                    <ion-button 
                      fill="clear" 
                      @click="copyText(viewingPassword.username)"
                      slot="end"
                    >
                      <ion-icon :icon="copyOutline"></ion-icon>
                    </ion-button>
                  </ion-item>

                  <ion-item v-if="viewingPassword.email">
                    <ion-label>
                      <h3>Email</h3>
                      <p>{{ viewingPassword.email }}</p>
                    </ion-label>
                    <ion-button 
                      fill="clear" 
                      @click="copyText(viewingPassword.email)"
                      slot="end"
                    >
                      <ion-icon :icon="copyOutline"></ion-icon>
                    </ion-button>
                  </ion-item>

                  <ion-item>
                    <ion-label>
                      <h3>Password</h3>
                      <p class="password-display">
                        {{ showViewPassword ? viewingPassword.password : '•'.repeat(viewingPassword.password.length) }}
                      </p>
                    </ion-label>
                    <ion-button 
                      fill="clear" 
                      @click="toggleViewPassword"
                      slot="end"
                    >
                      <ion-icon :icon="showViewPassword ? eyeOffOutline : eyeOutline"></ion-icon>
                    </ion-button>
                    <ion-button 
                      fill="clear" 
                      @click="copyText(viewingPassword.password)"
                      slot="end"
                    >
                      <ion-icon :icon="copyOutline"></ion-icon>
                    </ion-button>
                  </ion-item>

                  <ion-item v-if="viewingPassword.notes">
                    <ion-label>
                      <h3>Notes</h3>
                      <p>{{ viewingPassword.notes }}</p>
                    </ion-label>
                  </ion-item>
                </ion-list>
              </ion-card-content>
            </ion-card>

            <div class="view-meta">
              <p>Created: {{ formatDate(viewingPassword.createdAt) }}</p>
              <p>Last updated: {{ formatDate(viewingPassword.updatedAt) }}</p>
            </div>
          </div>
        </ion-content>
      </ion-modal>

      <!-- Floating Action Button -->
      <ion-fab v-if="passwords.length > 0" vertical="bottom" horizontal="end" slot="fixed">
        <ion-fab-button @click="openAddModal">
          <ion-icon :icon="addOutline"></ion-icon>
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons, IonButton,
  IonIcon, IonSearchbar, IonCard, IonCardContent, IonList, IonItem, IonLabel,
  IonInput, IonTextarea, IonSelect, IonSelectOption, IonCheckbox, IonChip,
  IonModal, IonSegment, IonSegmentButton, IonFab, IonFabButton
} from '@ionic/vue';
import {
  searchOutline, addOutline, lockClosedOutline, copyOutline, closeOutline,
  chevronForwardOutline, eyeOutline, eyeOffOutline, pencilOutline
} from 'ionicons/icons';

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

// State
const passwords = ref<Password[]>([]);
const showSearch = ref(false);
const searchQuery = ref('');
const selectedCategory = ref('all');
const isAddModalOpen = ref(false);
const isViewModalOpen = ref(false);
const editingPassword = ref<Password | null>(null);
const viewingPassword = ref<Password | null>(null);
const showPassword = ref(false);
const showViewPassword = ref(false);

// New password form
const newPassword = ref<Partial<Password>>({
  name: '',
  username: '',
  email: '',
  password: '',
  category: 'other',
  notes: ''
});

// Color palette
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

// Computed
const filteredPasswords = computed(() => {
  let filtered = passwords.value;
  
  // Filter by category
  if (selectedCategory.value !== 'all') {
    filtered = filtered.filter(p => p.category === selectedCategory.value);
  }
  
  // Filter by search query
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    filtered = filtered.filter(p => 
      p.name.toLowerCase().includes(query) ||
      (p.username?.toLowerCase() || '').includes(query) ||
      (p.email?.toLowerCase() || '').includes(query)
    );
  }
  
  return filtered.sort((a, b) => a.name.localeCompare(b.name));
});

// Methods
const openSearch = () => {
  showSearch.value = !showSearch.value;
  if (!showSearch.value) {
    searchQuery.value = '';
  }
};

const handleSearch = (event: CustomEvent) => {
  searchQuery.value = event.detail.value;
};

const filterByCategory = (event: CustomEvent) => {
  selectedCategory.value = event.detail.value;
};

const openAddModal = () => {
  editingPassword.value = null;
  newPassword.value = {
    name: '',
    username: '',
    email: '',
    password: '',
    category: 'other',
    notes: ''
  };
  showPassword.value = false;
  isAddModalOpen.value = true;
};

const openViewModal = (password: Password) => {
  viewingPassword.value = password;
  showViewPassword.value = false;
  isViewModalOpen.value = true;
};

const editPassword = (password: Password) => {
  editingPassword.value = password;
  newPassword.value = { ...password };
  showPassword.value = false;
  closeViewModal();
  isAddModalOpen.value = true;
};

const closeAddModal = () => {
  isAddModalOpen.value = false;
  editingPassword.value = null;
};

const closeViewModal = () => {
  isViewModalOpen.value = false;
  viewingPassword.value = null;
};

const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value;
};

const toggleViewPassword = () => {
  showViewPassword.value = !showViewPassword.value;
};

const generateId = () => {
  return Date.now().toString() + Math.random().toString(36).substr(2, 9);
};

const savePassword = () => {
  if (!newPassword.value.name || !newPassword.value.password) {
    return;
  }

  const now = new Date();
  
  if (editingPassword.value) {
    // Update existing password
    const index = passwords.value.findIndex(p => p.id === editingPassword.value!.id);
    if (index !== -1) {
      passwords.value[index] = {
        ...newPassword.value,
        id: editingPassword.value.id,
        createdAt: editingPassword.value.createdAt,
        updatedAt: now
      } as Password;
    }
  } else {
    // Add new password
    const password: Password = {
      id: generateId(),
      name: newPassword.value.name!,
      username: newPassword.value.username,
      email: newPassword.value.email,
      password: newPassword.value.password!,
      category: newPassword.value.category || 'other',
      notes: newPassword.value.notes,
      createdAt: now,
      updatedAt: now
    };
    
    passwords.value.push(password);
  }
  
  saveToStorage();
  closeAddModal();
};

const deletePassword = () => {
  if (editingPassword.value) {
    const index = passwords.value.findIndex(p => p.id === editingPassword.value!.id);
    if (index !== -1) {
      passwords.value.splice(index, 1);
      saveToStorage();
      closeAddModal();
    }
  }
};

const copyPassword = async (password: string) => {
  await copyText(password);
};

const copyText = async (text: string) => {
  try {
    await navigator.clipboard.writeText(text);
    // You could add a toast notification here
    console.log('Copied to clipboard');
  } catch (error) {
    console.error('Failed to copy:', error);
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

const saveToStorage = () => {
  try {
    localStorage.setItem('vault-passwords', JSON.stringify(passwords.value));
  } catch (error) {
    console.error('Failed to save to storage:', error);
  }
};

const loadFromStorage = () => {
  try {
    const stored = localStorage.getItem('vault-passwords');
    if (stored) {
      const parsed = JSON.parse(stored);
      // Convert date strings back to Date objects
      passwords.value = parsed.map((p: any) => ({
        ...p,
        createdAt: new Date(p.createdAt),
        updatedAt: new Date(p.updatedAt)
      }));
    }
  } catch (error) {
    console.error('Failed to load from storage:', error);
  }
};

onMounted(() => {
  loadFromStorage();
});
</script>

<style scoped>
.search-container {
  padding: 0 16px;
  background: var(--ion-background-color);
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 40px;
  text-align: center;
  min-height: 60vh;
}

.empty-icon {
  font-size: 100px;
  color: var(--ion-color-medium);
  margin-bottom: 24px;
}

.empty-state h2 {
  font-size: 24px;
  font-weight: 600;
  color: var(--ion-text-color);
  margin: 0 0 12px 0;
}

.empty-state p {
  font-size: 16px;
  color: var(--ion-color-medium);
  margin: 0 0 32px 0;
  line-height: 1.4;
}

.add-first-button {
  --border-radius: 12px;
  margin: 0;
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

.vault-content {
  padding: 0 16px 100px 16px;
}

.categories {
  margin: 16px 0;
}

.passwords-container {
  margin-top: 16px;
}

.password-card {
  margin-bottom: 12px;
  border-radius: 16px;
}

.password-item {
  display: flex;
  align-items: center;
  gap: 16px;
}

.password-avatar {
  flex-shrink: 0;
}

.app-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 20px;
}

.app-icon.large {
  width: 80px;
  height: 80px;
  font-size: 32px;
  border-radius: 20px;
}

.password-info {
  flex: 1;
  min-width: 0;
}

.password-info h3 {
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 4px 0;
  color: var(--ion-text-color);
}

.password-info p {
  font-size: 14px;
  color: var(--ion-color-medium);
  margin: 0 0 8px 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.password-meta {
  display: flex;
  align-items: center;
  gap: 8px;
}

.last-updated {
  font-size: 12px;
  color: var(--ion-color-medium);
}

.password-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.copy-btn {
  --color: var(--ion-color-primary);
}

.save-button {
  margin: 24px 0 16px 0;
  --border-radius: 12px;
}

.delete-button {
  margin: 0 0 16px 0;
  --border-radius: 12px;
}

.view-content {
  padding-bottom: 40px;
}

.view-header {
  text-align: center;
  margin-bottom: 24px;
}

.view-avatar {
  margin-bottom: 16px;
}

.view-header h2 {
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

.view-meta {
  margin-top: 24px;
  text-align: center;
}

.view-meta p {
  font-size: 14px;
  color: var(--ion-color-medium);
  margin: 4px 0;
}
</style>