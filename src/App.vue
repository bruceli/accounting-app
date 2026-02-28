<template>
  <div class="app-container" :class="theme">
    <!-- Header -->
    <header class="header">
      <h1>记账本</h1>
      <button class="theme-toggle" @click="toggleTheme">
        {{ theme === 'dark' ? '☀️' : '🌙' }}
      </button>
      <div class="date-display">{{ currentDate }}</div>
    </header>

    <!-- Summary Cards -->
    <div class="summary-cards">
      <div class="card balance">
        <span class="label">余额</span>
        <span class="amount">¥{{ formatMoney(balance) }}</span>
      </div>
      <div class="card income">
        <span class="label">收入</span>
        <span class="amount">+¥{{ formatMoney(totalIncome) }}</span>
      </div>
      <div class="card expense">
        <span class="label">支出</span>
        <span class="amount">-¥{{ formatMoney(totalExpense) }}</span>
      </div>
    </div>

    <!-- Quick Add -->
    <div class="quick-add">
      <button class="btn-income" @click="showAddModal('income')">+ 收入</button>
      <button class="btn-expense" @click="showAddModal('expense')">+ 支出</button>
    </div>

    <!-- Transaction List -->
    <div class="transaction-section">
      <h2>交易记录</h2>
      <div class="transaction-list" v-if="transactions.length">
        <div 
          v-for="tx in transactions" 
          :key="tx.id" 
          class="transaction-item"
          :class="tx.type"
        >
          <div class="tx-icon">{{ tx.categoryIcon }}</div>
          <div class="tx-details">
            <div class="tx-category">{{ tx.category }}</div>
            <div class="tx-note">{{ tx.note || '无备注' }}</div>
            <div class="tx-date">{{ formatDate(tx.date) }}</div>
          </div>
          <div class="tx-amount" :class="tx.type">
            {{ tx.type === 'income' ? '+' : '-' }}¥{{ formatMoney(tx.amount) }}
          </div>
          <button class="delete-btn" @click="deleteTx(tx.id)">×</button>
        </div>
      </div>
      <div v-else class="empty-state">
        暂无记录，点击上方按钮添加第一笔交易
      </div>
    </div>

    <!-- Add Modal -->
    <div class="modal" v-if="showModal" @click.self="closeModal">
      <div class="modal-content">
        <h3>{{ modalType === 'income' ? '添加收入' : '添加支出' }}</h3>
        
        <div class="form-group">
          <label>金额</label>
          <input 
            type="number" 
            v-model.number="newTx.amount" 
            placeholder="0.00"
            step="0.01"
          />
        </div>

        <div class="form-group">
          <label>分类</label>
          <div class="category-grid">
            <button 
              v-for="cat in getCategories(modalType)" 
              :key="cat.name"
              class="category-btn"
              :class="{ active: newTx.category === cat.name }"
              @click="newTx.category = cat.name"
            >
              <span class="cat-icon">{{ cat.icon }}</span>
              <span>{{ cat.name }}</span>
            </button>
          </div>
        </div>

        <div class="form-group">
          <label>备注</label>
          <input 
            type="text" 
            v-model="newTx.note" 
            placeholder="可选备注"
          />
        </div>

        <div class="form-group">
          <label>日期</label>
          <input 
            type="date" 
            v-model="newTx.date" 
          />
        </div>

        <div class="modal-actions">
          <button class="btn-cancel" @click="closeModal">取消</button>
          <button 
            class="btn-confirm" 
            :class="modalType"
            @click="addTransaction"
          >
            确定
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      transactions: [],
      showModal: false,
      modalType: 'expense',
      theme: 'light',
      newTx: {
        amount: '',
        category: '',
        note: '',
        date: new Date().toISOString().split('T')[0]
      }
    }
  },
  computed: {
    currentDate() {
      const now = new Date();
      return now.toLocaleDateString('zh-CN', { 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric',
        weekday: 'long'
      });
    },
    totalIncome() {
      return this.transactions
        .filter(t => t.type === 'income')
        .reduce((sum, t) => sum + t.amount, 0);
    },
    totalExpense() {
      return this.transactions
        .filter(t => t.type === 'expense')
        .reduce((sum, t) => sum + t.amount, 0);
    },
    balance() {
      return this.totalIncome - this.totalExpense;
    }
  },
  methods: {
    getCategories(type) {
      const categories = {
        income: [
          { name: '工资', icon: '💰' },
          { name: '奖金', icon: '🎁' },
          { name: '投资', icon: '📈' },
          { name: '兼职', icon: '💼' },
          { name: '其他收入', icon: '💵' }
        ],
        expense: [
          { name: '餐饮', icon: '🍔' },
          { name: '交通', icon: '🚗' },
          { name: '购物', icon: '🛒' },
          { name: '居住', icon: '🏠' },
          { name: '娱乐', icon: '🎮' },
          { name: '医疗', icon: '💊' },
          { name: '教育', icon: '📚' },
          { name: '其他', icon: '📦' }
        ]
      };
      return categories[type];
    },
    showAddModal(type) {
      this.modalType = type;
      const cats = this.getCategories(type);
      this.newTx = {
        amount: '',
        category: cats[0].name,
        note: '',
        date: new Date().toISOString().split('T')[0]
      };
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
    addTransaction() {
      if (!this.newTx.amount || this.newTx.amount <= 0) {
        alert('请输入有效金额');
        return;
      }
      if (!this.newTx.category) {
        alert('请选择分类');
        return;
      }

      const cats = this.getCategories(this.modalType);
      const cat = cats.find(c => c.name === this.newTx.category);

      const tx = {
        id: Date.now(),
        type: this.modalType,
        amount: this.newTx.amount,
        category: this.newTx.category,
        categoryIcon: cat ? cat.icon : '💰',
        note: this.newTx.note,
        date: this.newTx.date
      };

      this.transactions.unshift(tx);
      this.saveToStorage();
      this.closeModal();
    },
    deleteTx(id) {
      if (confirm('确定删除这条记录？')) {
        this.transactions = this.transactions.filter(t => t.id !== id);
        this.saveToStorage();
      }
    },
    formatMoney(amount) {
      return Number(amount).toFixed(2);
    },
    formatDate(dateStr) {
      const date = new Date(dateStr);
      const today = new Date();
      const yesterday = new Date(today);
      yesterday.setDate(yesterday.getDate() - 1);

      if (dateStr === today.toISOString().split('T')[0]) {
        return '今天';
      } else if (dateStr === yesterday.toISOString().split('T')[0]) {
        return '昨天';
      }
      return date.toLocaleDateString('zh-CN', { month: 'numeric', day: 'numeric' });
    },
    saveToStorage() {
      localStorage.setItem('accounting-transactions', JSON.stringify(this.transactions));
    },
    loadFromStorage() {
      const saved = localStorage.getItem('accounting-transactions');
      if (saved) {
        this.transactions = JSON.parse(saved);
      }
    },
    toggleTheme() {
      this.theme = this.theme === 'light' ? 'dark' : 'light';
      localStorage.setItem('accounting-theme', this.theme);
    },
    loadTheme() {
      const saved = localStorage.getItem('accounting-theme');
      if (saved) {
        this.theme = saved;
      }
    }
  },
  mounted() {
    this.loadFromStorage();
    this.loadTheme();
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg-primary: #f5f5f5;
  --bg-secondary: white;
  --text-primary: #333;
  --text-secondary: #999;
  --text-muted: #bbb;
  --border-color: #f0f0f0;
  --card-shadow: 0 2px 8px rgba(0,0,0,0.06);
  --input-bg: white;
  --input-border: #ddd;
}

.dark {
  --bg-primary: #1a1a1a;
  --bg-secondary: #2a2a2a;
  --text-primary: #e0e0e0;
  --text-secondary: #888;
  --text-muted: #666;
  --border-color: #3a3a3a;
  --card-shadow: 0 2px 8px rgba(0,0,0,0.3);
  --input-bg: #333;
  --input-border: #444;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: var(--bg-primary);
  min-height: 100vh;
  transition: background 0.3s;
}

.app-container {
  max-width: 480px;
  margin: 0 auto;
  padding: 20px;
  padding-bottom: 100px;
}

.header {
  text-align: center;
  margin-bottom: 20px;
  position: relative;
}

.header h1 {
  font-size: 24px;
  color: var(--text-primary);
  margin-bottom: 8px;
}

.theme-toggle {
  position: absolute;
  right: 0;
  top: 0;
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 8px 12px;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s;
}

.date-display {
  font-size: 14px;
  color: var(--text-secondary);
}

.summary-cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin-bottom: 20px;
}

.card {
  background: var(--bg-secondary);
  border-radius: 12px;
  padding: 16px 12px;
  text-align: center;
  box-shadow: var(--card-shadow);
  transition: background 0.3s;
}

.card.balance {
  grid-column: span 3;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.card.balance .label,
.card.balance .amount {
  color: white;
}

.card.income .amount {
  color: #52c41a;
}

.card.expense .amount {
  color: #ff4d4f;
}

.card .label {
  display: block;
  font-size: 12px;
  color: var(--text-secondary);
  margin-bottom: 4px;
}

.card .amount {
  font-size: 18px;
  font-weight: 600;
}

.quick-add {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-bottom: 24px;
}

.quick-add button {
  border: none;
  padding: 16px;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.quick-add button:active {
  transform: scale(0.98);
}

.btn-income {
  background: #f6ffed;
  color: #52c41a;
  border: 1px solid #b7eb8f !important;
}

.btn-expense {
  background: #fff1f0;
  color: #ff4d4f;
  border: 1px solid #ffccc7 !important;
}

.transaction-section h2 {
  font-size: 18px;
  color: var(--text-primary);
  margin-bottom: 12px;
}

.transaction-list {
  background: var(--bg-secondary);
  border-radius: 12px;
  overflow: hidden;
  box-shadow: var(--card-shadow);
  transition: background 0.3s;
}

.transaction-item {
  display: flex;
  align-items: center;
  padding: 14px 16px;
  border-bottom: 1px solid var(--border-color);
  transition: background 0.3s, border-color 0.3s;
}

.transaction-item:last-child {
  border-bottom: none;
}

.tx-icon {
  font-size: 24px;
  margin-right: 12px;
}

.tx-details {
  flex: 1;
}

.tx-category {
  font-size: 15px;
  color: var(--text-primary);
  font-weight: 500;
}

.tx-note {
  font-size: 13px;
  color: var(--text-secondary);
  margin-top: 2px;
}

.tx-date {
  font-size: 12px;
  color: var(--text-muted);
  margin-top: 2px;
}

.tx-amount {
  font-size: 16px;
  font-weight: 600;
  margin-right: 8px;
}

.tx-amount.income {
  color: #52c41a;
}

.tx-amount.expense {
  color: #ff4d4f;
}

.delete-btn {
  background: none;
  border: none;
  font-size: 20px;
  color: var(--text-muted);
  cursor: pointer;
  padding: 4px 8px;
}

.delete-btn:hover {
  color: #ff4d4f;
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: var(--text-secondary);
  background: var(--bg-secondary);
  border-radius: 12px;
  transition: background 0.3s, color 0.3s;
}

/* Modal */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: flex-end;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: var(--bg-secondary);
  border-radius: 20px 20px 0 0;
  padding: 24px;
  width: 100%;
  max-width: 480px;
  max-height: 85vh;
  overflow-y: auto;
  animation: slideUp 0.3s ease;
  transition: background 0.3s;
}

@keyframes slideUp {
  from {
    transform: translateY(100%);
  }
  to {
    transform: translateY(0);
  }
}

.modal-content h3 {
  text-align: center;
  font-size: 18px;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  font-size: 14px;
  color: var(--text-secondary);
  margin-bottom: 8px;
}

.form-group input {
  width: 100%;
  padding: 12px 16px;
  border: 1px solid var(--input-border);
  border-radius: 8px;
  font-size: 16px;
  outline: none;
  transition: border-color 0.2s, background 0.3s;
  background: var(--input-bg);
  color: var(--text-primary);
}

.form-group input:focus {
  border-color: #667eea;
}

.category-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.category-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px 4px;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  background: var(--bg-secondary);
  cursor: pointer;
  transition: all 0.2s;
}

.category-btn.active {
  border-color: #667eea;
  background: #f0f0ff;
}

.cat-icon {
  font-size: 20px;
  margin-bottom: 4px;
}

.category-btn span:last-child {
  font-size: 12px;
  color: var(--text-secondary);
}

.modal-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-top: 24px;
}

.btn-cancel,
.btn-confirm {
  padding: 14px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  border: none;
}

.btn-cancel {
  background: var(--bg-primary);
  color: var(--text-secondary);
}

.btn-confirm {
  color: white;
}

.btn-confirm.income {
  background: #52c41a;
}

.btn-confirm.expense {
  background: #ff4d4f;
}
</style>
