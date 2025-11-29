<template>
  <div class="payment-system">
    <!-- Header -->
    <header class="header">
      <h1>🏫 Sistema de Pagos Escolares</h1>
      <p>Procesamiento real con Stripe</p>
    </header>

    <div class="container">
      <!-- Información del Estudiante -->
      <div class="student-card">
        <h2>👨‍🎓 Información del Estudiante</h2>
        <div class="student-info">
          <p><strong>Nombre:</strong> {{ student.name }}</p>
          <p><strong>Email:</strong> {{ student.email }}</p>
          <p><strong>Carrera:</strong> {{ student.career }}</p>
          <p><strong>Saldo Disponible:</strong> ${{ student.balance }}</p>
        </div>
      </div>

      <!-- Formulario de Pago -->
      <div class="payment-card">
        <h2>💳 Realizar Pago</h2>
        
        <div class="form-group">
          <label>Concepto de Pago:</label>
          <input 
            v-model="paymentData.description" 
            placeholder="Ej: Colegiatura, Materiales, Examen..."
            class="form-input"
          >
        </div>

        <div class="form-group">
          <label>Monto a Pagar:</label>
          <input 
            v-model="paymentData.amount" 
            type="number" 
            min="1"
            :max="student.balance"
            placeholder="0.00"
            class="form-input"
          >
          <small>Máximo: ${{ student.balance }}</small>
        </div>

        <div class="form-group">
          <label>Seleccionar Tarjeta de Prueba:</label>
          <select v-model="paymentData.selectedCard" class="form-input">
            <option 
              v-for="card in testCards" 
              :key="card.id" 
              :value="card"
            >
              {{ card.name }} - {{ card.number }}
            </option>
          </select>
          <small>Tarjetas proporcionadas por Stripe para testing</small>
        </div>

        <button 
          @click="processPayment" 
          :disabled="processing || !isFormValid"
          class="pay-button"
        >
          {{ processing ? '🔄 Procesando...' : '🚀 Pagar con Stripe' }}
        </button>

        <div v-if="error" class="error-message">
          {{ error }}
        </div>
      </div>

      <!-- Resultado del Pago -->
      <div v-if="paymentResult" class="result-card">
        <h2>✅ Pago Completado</h2>
        <div class="transaction-details">
          <p><strong>ID de Transacción:</strong> {{ paymentResult.paymentId }}</p>
          <p><strong>Concepto:</strong> {{ paymentData.description }}</p>
          <p><strong>Monto Pagado:</strong> ${{ paymentData.amount }}</p>
          <p><strong>Nuevo Saldo:</strong> ${{ paymentResult.student.newBalance }}</p>
          <p><strong>Fecha:</strong> {{ new Date().toLocaleString() }}</p>
          <p><strong>Método:</strong> Stripe ({{ paymentData.selectedCard.network }})</p>
        </div>
      </div>

      <!-- Tarjetas de Prueba -->
      <div class="test-cards-info">
        <h3>🃏 Tarjetas de Prueba Stripe</h3>
        <div class="cards-grid">
          <div 
            v-for="card in testCards" 
            :key="card.id"
            class="card-item"
            @click="paymentData.selectedCard = card"
          >
            <div class="card-icon">{{ card.icon }}</div>
            <div class="card-details">
              <strong>{{ card.name }}</strong>
              <div>{{ card.number }}</div>
              <small>{{ card.description }}</small>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PaymentSystem',
  data() {
    return {
      student: {
        id: 1,
        name: 'Ana García',
        email: 'ana@escuela.com',
        balance: 1500.00,
        career: 'Ingeniería de Software'
      },
      paymentData: {
        amount: 0,
        description: 'Colegiatura mensual',
        selectedCard: null
      },
      processing: false,
      error: '',
      paymentResult: null,
      testCards: [
        {
          id: 'visa_success',
          name: 'Visa',
          number: '4242424242424242',
          payment_method: 'pm_card_visa',
          description: 'Pago siempre exitoso',
          network: 'visa',
          icon: '💳'
        },
        {
          id: 'visa_decline', 
          name: 'Visa (Rechazada)',
          number: '4000000000000002',
          payment_method: 'pm_card_visa_chargeDeclined',
          description: 'Siempre rechazada',
          network: 'visa',
          icon: '❌'
        },
        {
          id: 'mastercard',
          name: 'Mastercard', 
          number: '5555555555554444',
          payment_method: 'pm_card_mastercard',
          description: 'Pago exitoso',
          network: 'mastercard',
          icon: '💳'
        },
        {
          id: 'insufficient_funds',
          name: 'Fondos Insuficientes',
          number: '4000000000009995',
          payment_method: 'pm_card_visa_chargeDeclinedInsufficientFunds',
          description: 'Fondos insuficientes',
          network: 'visa',
          icon: '💰'
        }
      ]
    }
  },
  computed: {
    isFormValid() {
      return this.paymentData.amount > 0 && 
             this.paymentData.amount <= this.student.balance &&
             this.paymentData.selectedCard;
    }
  },
  mounted() {
    this.paymentData.selectedCard = this.testCards[0];
  },
  methods: {
    async processPayment() {
      this.processing = true;
      this.error = '';
      this.paymentResult = null;

      try {
        const response = await fetch('http://localhost:3000/api/process-payment', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            studentId: this.student.id,
            amount: parseFloat(this.paymentData.amount),
            paymentMethod: this.paymentData.selectedCard.payment_method,
            description: this.paymentData.description
          })
        });

        const result = await response.json();
        
        if (result.success) {
          this.paymentResult = result;
          this.student.balance = result.student.newBalance;
          this.$nextTick(() => {
            window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
          });
        } else {
          this.error = result.error || 'Error desconocido';
        }
      } catch (err) {
        this.error = 'Error de conexión con el servidor';
        console.error('Error:', err);
      } finally {
        this.processing = false;
      }
    }
  }
}
</script>

<style scoped>
.payment-system {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.header {
  text-align: center;
  margin-bottom: 30px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 30px;
  border-radius: 15px;
}

.container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.student-card, .payment-card, .result-card, .test-cards-info {
  background: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  border: 1px solid #e1e5e9;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2d3748;
}

.form-input {
  width: 100%;
  padding: 12px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.form-input:focus {
  outline: none;
  border-color: #667eea;
}

.pay-button {
  width: 100%;
  padding: 15px;
  background: linear-gradient(135deg, #48bb78 0%, #38a169 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 18px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s;
}

.pay-button:hover:not(:disabled) {
  transform: translateY(-2px);
}

.pay-button:disabled {
  background: #a0aec0;
  cursor: not-allowed;
  transform: none;
}

.error-message {
  background: #fed7d7;
  color: #c53030;
  padding: 12px;
  border-radius: 8px;
  margin-top: 15px;
  border-left: 4px solid #f56565;
}

.transaction-details {
  background: #f0fff4;
  padding: 20px;
  border-radius: 8px;
  border-left: 4px solid #48bb78;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
  margin-top: 15px;
}

.card-item {
  display: flex;
  align-items: center;
  padding: 15px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.card-item:hover {
  border-color: #667eea;
  transform: translateY(-2px);
}

.card-item.active {
  border-color: #667eea;
  background: #f7fafc;
}

.card-icon {
  font-size: 24px;
  margin-right: 15px;
}

.card-details {
  flex: 1;
}

.card-details small {
  color: #718096;
}

@media (max-width: 768px) {
  .payment-system {
    padding: 10px;
  }
  
  .cards-grid {
    grid-template-columns: 1fr;
  }
}
</style>
