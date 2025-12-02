<template>
  <nav class="navbar">
    <div class="nav-brand">Dashboard A3</div>
    <div class="nav-links">
      <button 
        v-for="tab in tabs" 
        :key="tab.id"
        :class="['nav-item', { active: currentTab === tab.id }]"
        @click="$emit('change-tab', tab.id)"
      >
        {{ tab.label }}
      </button>
    </div>
  </nav>
</template>

<script setup>
defineProps({
  currentTab: {
    type: String,
    required: true
  },
  tabs: {
    type: Array,
    required: true
  }
})

defineEmits(['change-tab'])
</script>

<style scoped>
.navbar {
  background-color: #1a1a1a;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  margin-bottom: 2rem;
}

.nav-brand {
  font-size: 1.5rem;
  font-weight: bold;
  color: #41B883;
}

.nav-links {
  display: flex;
  gap: 1rem;
}

.nav-item {
  background: transparent;
  border: none;
  color: #e0e0e0;
  padding: 0.5rem 1rem;
  cursor: pointer;
  font-size: 1rem;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.nav-item:hover {
  background-color: #333;
  color: #fff;
}

.nav-item.active {
  background-color: #41B883;
  color: #1a1a1a;
  font-weight: bold;
}

@media (max-width: 768px) {
  .navbar {
    flex-direction: column;
    gap: 1rem;
  }
  
  .nav-links {
    flex-wrap: wrap;
    justify-content: center;
  }
}
</style>
