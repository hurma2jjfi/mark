<template>
  <div v-if="showCalculator" class="center">
    <div class="grid">
      <button @click="toggleCalculator" class="close-button" title="Закрыть">X</button>
      <input @keyup.enter="calc()" type="text" v-model="result" placeholder="0" id="calc__field">
      <button @click="input(num)" class="cell num" v-for="num in numbers" :key="num">{{num}}</button>
      <button @click="input(op)" class="cell op" v-for="op in operations" :key="op">{{op}}</button>
      <button @click="input(',')" class="cell op">,</button> 
      <!-- <button @click="input('**')" class="cell op">^</button> -->
      <button @click="reset()" class="cell op" title="Стереть">R</button>
      <button @click="calc()" class="cell op" id="yellow" title="Считать">=</button>
      <div v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showCalculator: true,
      result: '',
      errorMessage: '', 
      numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, 0],
      operations: ['+', '-', '*', '/'],
    };
  },
  methods: {
    input(char) {
      if (this.errorMessage) {
        this.reset(); 
      }
      this.result += char; 
      this.errorMessage = ''; 
    },
    reset() {
      this.result = '';
      this.errorMessage = ''; 
    },
    calc() {
      try {
        if (typeof this.result !== 'string') {
          throw new Error('Результат не должен быть строкой.');
        }

        const trimmedResult = this.result.trim();
        
        if (trimmedResult === '') {
          throw new Error('Пожалуйста введите выражение, например 2+2');
        }

        const operatorOnlyRegex = /^[+\-*/\s]+$/;

        if (operatorOnlyRegex.test(trimmedResult)) {
          throw new Error('Введены только операторы. Пожалуйста, введите число.');
        }

        const consecutiveOperatorsRegex = /[+\-*\/]{2,}|(\*\*{2,})|([+\-*\/]\*\*)|(\*\*[+\-*\/])/;
        if (consecutiveOperatorsRegex.test(trimmedResult)) {
          throw new Error('Недопустимы последовательные операторы.');
        }

       
        const leadingZerosRegex = /(?<!\d)0\d+|\.\d+|(?<=\s|^)\d+\.0+/g;
        if (leadingZerosRegex.test(trimmedResult)) {
          throw new Error('Недопустимые нули в числе.');
        }

        const sanitizedResult = trimmedResult.replace(/[^0-9+\-*/().**,]/g, '');

        const safeEval = (expression) => {
          expression = expression.replace(/,/g, '.');
          const parts = expression.split(/([\+\-\*\/\*\*])/); 
          for (let i = 0; i < parts.length; i++) {
            if (parts[i] === '/') {
              const nextNum = parseFloat(parts[i + 1]);
              if (nextNum === 0) {
                throw new Error('Нельзя делить на ноль!');
              }
            }
          }
          return eval(expression);
        };

    
        this.result = safeEval(sanitizedResult).toString();
        this.errorMessage = ''; 
      } catch (error) {
        this.errorMessage = error.message; 
        this.result = ''; 
      }
    },
    toggleCalculator() {
      this.showCalculator = !this.showCalculator;
    },
  },
};
</script>

<style>

* {
    margin: 0;
    padding: 0;
    font-family:Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
    transition: 0.3s all;
    font-weight: 500;
}

#calc__field {
    position: relative;
    top: -5px;
}

#app {
    width: 250px; 
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.2); 
    background-color: #f9f9f9; 
    border-radius: 10px; 
    padding: 20px; 
    display: flex;
    flex-direction: column;
    align-items: center;
}

.close-button {
    margin-left: auto; 
    margin-top: 20px; 
    margin-bottom: 10px; 
    background-color: #e74c3c;
    color: #fff;
    border: none;
    padding: 8px 12px; 
    cursor: pointer;
    font-size: 14px;
    border-radius: 5px;
}



input[type="text"] {
    width: calc(100%); 
    border: 1px solid #ccc; 
    color: #333;
    font-size: 20px; 
    padding: 10px; 
    text-align: right;
   outline: none;
}

.grid {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between; 
    width: 100%;
}

.cell {
    flex-basis: calc(33% - 5px); 
    height: 60px; 
    font-size: 18px;
}

.num {
    background-color: #3498db;
    color: #fff;
}

.num, .cell {
   border: none;
}

.cell:hover {
   background-color: #2980b9; 
}

.op {
   background-color: #f39c12; 
   color: #fff;
}


.error-message {
   color: red; 
   margin-top: 10px; 
   padding: 10px; 
   border-radius: 5px; 
   background-color: rgba(255,0,0,0.1); 
   width: calc(100%); 
   text-align: center; 
   font-size: 14px;
}

</style>