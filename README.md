# Dashboard A3

Este projeto é um dashboard Vue.js que consome dados da API [https://a3-la3z.onrender.com](https://a3-la3z.onrender.com).

## Pré-requisitos

- Node.js instalado

## Como rodar

1. Abra o terminal na pasta `FrontEnd`.
2. Instale as dependências (se ainda não instalou):
   ```bash
   npm install
   ```
3. Rode o servidor de desenvolvimento:
   ```bash
   npm run dev
   ```
4. Abra o navegador no link mostrado no terminal (geralmente http://localhost:5173).

## Estrutura

- `src/components/Dashboard.vue`: Componente principal que busca os dados e exibe os gráficos.
- `src/components/BarChart.vue`: Componente de gráfico de barras.
- `src/components/PieChart.vue`: Componente de gráfico de pizza.
