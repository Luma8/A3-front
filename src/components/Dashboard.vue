<template>
    <div class="dashboard-container">
        <Navbar :currentTab="currentTab" :tabs="tabs" @change-tab="currentTab = $event" />
        
        <div class="filters-container">
            <label for="gender-filter">Filtrar por Gênero:</label>
            <select id="gender-filter" v-model="selectedGender" @change="fetchData">
                <option value="">Todos</option>
                <option value="Homem">Homem</option>
                <option value="Mulher">Mulher</option>
                <option value="Não binário">Não binário</option>
            </select>
        </div>

        <div class="dashboard">
            <div v-if="loading" class="loading">Carregando dados...</div>
            <div v-else-if="error" class="error">{{ error }}</div>
            <div v-else-if="stats && stats.message" class="no-data">{{ stats.message }}</div>

            <div v-else class="content">
                <div class="summary-cards">
                    <div class="card">
                        <h3>Total de Participantes</h3>
                        <p class="big-number">{{ stats.total_participantes }}</p>
                    </div>
                    <div class="card">
                        <h3>Média de Idade</h3>
                        <p class="big-number">{{ stats.media_idade }}</p>
                        <p class="small-text">anos</p>
                    </div>
                    <div class="card">
                        <h3>Desvio Padrão</h3>
                        <p class="big-number">{{ stats.desvio_padrao_idade }}</p>
                        <p class="small-text">anos</p>
                    </div>
                </div>

                <div v-if="currentTab === 'geral'" class="charts-grid">
                    <div class="chart-container">
                        <h3>Gênero</h3>
                        <div class="chart-wrapper">
                            <PieChart v-if="genderData" :chartData="genderData" />
                        </div>
                    </div>
                    <div class="chart-container">
                        <h3>Avaliação Geral</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="generalEvalData" :chartData="generalEvalData" />
                        </div>
                    </div>
                </div>

                <div v-if="currentTab === 'escolaridade'" class="charts-grid">
                    <div class="chart-container">
                        <h3>Ano Escolar</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="schoolYearData" :chartData="schoolYearData" />
                        </div>
                    </div>
                </div>

                <div v-if="currentTab === 'pretensao'" class="charts-grid">
                    <div class="chart-container">
                        <h3>Universidade Pretendida</h3>
                        <div class="chart-wrapper">
                            <PieChart v-if="universityData" :chartData="universityData" />
                        </div>
                    </div>
                </div>

                <div v-if="currentTab === 'materias'" class="charts-grid">
                    <div class="chart-container">
                        <h3>Interesse em Tecnologia</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="techInterestData" :chartData="techInterestData" />
                        </div>
                    </div>
                    <div class="chart-container">
                        <h3>Interesse em Desafios</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="challengesInterestData" :chartData="challengesInterestData" />
                        </div>
                    </div>
                    <div class="chart-container">
                        <h3>Interesse em Matemática</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="mathInterestData" :chartData="mathInterestData" />
                        </div>
                    </div>
                    <div class="chart-container">
                        <h3>Interesse em Português</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="portugueseInterestData" :chartData="portugueseInterestData" />
                        </div>
                    </div>
                    <div class="chart-container full-width">
                        <h3>Matéria Preferida</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="favoriteSubjectData" :chartData="favoriteSubjectData" />
                        </div>
                    </div>
                </div>

                <div v-if="currentTab === 'tecnologia'" class="charts-grid">
                    <div class="chart-container">
                        <h3>Turno de Preferência</h3>
                        <div class="chart-wrapper">
                            <PieChart v-if="shiftPreferenceData" :chartData="shiftPreferenceData" />
                        </div>
                    </div>
                    <div class="chart-container">
                        <h3>Contato com Programação</h3>
                        <div class="chart-wrapper">
                            <PieChart v-if="programmingContactData" :chartData="programmingContactData" />
                        </div>
                    </div>
                    <div class="chart-container">
                        <h3>Gosta de Jogos Eletrônicos</h3>
                        <div class="chart-wrapper">
                            <PieChart v-if="likesGamesData" :chartData="likesGamesData" />
                        </div>
                    </div>
                    <div class="chart-container full-width">
                        <h3>Acesso a Dispositivos e Internet</h3>
                        <div class="chart-wrapper">
                            <BarChart v-if="devicesData" :chartData="devicesData" />
                        </div>
                    </div>
                </div>

                <div v-if="currentTab === 'perfis'" class="profiles-section">
                    <h2 class="section-title">Foco em Tecnologia</h2>
                    <div class="profiles-grid">
                        <div class="profile-card" v-for="key in profileKeys" :key="'tech-'+key">
                            <h4>{{ formatProfileTitle(key) }}</h4>
                            <div v-if="stats.perfis_especificos.tech[key]">
                                <p><strong>Alunos:</strong> {{ stats.perfis_especificos.tech[key].count }}</p>
                                <p><strong>Turno:</strong> {{ stats.perfis_especificos.tech[key].top_turno }}</p>
                                <p><strong>Matéria:</strong> {{ stats.perfis_especificos.tech[key].top_materia }}</p>
                                <p><strong>Já Programou:</strong> {{ stats.perfis_especificos.tech[key].pct_contato }}%</p>
                            </div>
                            <div v-else class="no-profile-data">Sem dados</div>
                        </div>
                    </div>

                    <h2 class="section-title">Foco em Humanas</h2>
                    <div class="profiles-grid">
                        <div class="profile-card" v-for="key in profileKeys" :key="'humanas-'+key">
                            <h4>{{ formatProfileTitle(key) }}</h4>
                            <div v-if="stats.perfis_especificos.humanas[key]">
                                <p><strong>Alunos:</strong> {{ stats.perfis_especificos.humanas[key].count }}</p>
                                <p><strong>Turno:</strong> {{ stats.perfis_especificos.humanas[key].top_turno }}</p>
                                <p><strong>Matéria:</strong> {{ stats.perfis_especificos.humanas[key].top_materia }}</p>
                                <p><strong>Já Programou:</strong> {{ stats.perfis_especificos.humanas[key].pct_contato }}%</p>
                            </div>
                            <div v-else class="no-profile-data">Sem dados</div>
                        </div>
                    </div>
                </div>

                <!-- Tab: Análise Interpretativa -->
                <div v-if="currentTab === 'analise'" class="analysis-section">
                    <div class="analysis-card">
                        <h2 class="section-title">Análise do Perfil dos Participantes</h2>
                        <div class="analysis-text">
                            <p>
                                A análise dos dados coletados no evento "Vem para o Mundo" revela insights importantes sobre o perfil dos estudantes e seu potencial interesse na área de Computação.
                            </p>
                            
                            <h3>1. Perfil Demográfico e Idade</h3>
                            <p>
                                O público é majoritariamente jovem, com uma média de idade de <strong>{{ stats.media_idade }} anos</strong>. 
                                O desvio padrão de <strong>{{ stats.desvio_padrao_idade }}</strong> indica uma {{ stats.desvio_padrao_idade < 1.5 ? 'baixa' : 'moderada' }} dispersão, 
                                confirmando que o evento atingiu seu público-alvo principal: alunos do ensino médio em fase de decisão de carreira.
                            </p>

                            <h3>2. Interesse por Tecnologia</h3>
                            <p>
                                Observa-se uma correlação clara entre o gênero e o interesse declarado em tecnologia. 
                                Os dados da aba "Perfis" mostram que os alunos do gênero masculino tendem a ter um interesse prévio mais consolidado ("Muito Interesse"), 
                                enquanto há um campo de oportunidade significativo para engajar mais alunas, que muitas vezes aparecem com interesses divididos ou focados em outras áreas.
                            </p>

                            <h3>3. Preferências Acadêmicas e Interdisciplinaridade</h3>
                            <p>
                                As matérias preferidas (Matemática vs. Português/Humanas) alinham-se fortemente com a escolha do curso. 
                                Alunos que indicam "Matemática" como favorita são os que mais demonstram interesse imediato em Ciência da Computação.
                            </p>
                            <p>
                                Um dado curioso é a presença de "História" como matéria favorita em alguns perfis com alto interesse tecnológico. 
                                Isso é justificado pelo fato de que uma parcela significativa desses alunos já teve contato prático com programação (como evidenciado na métrica "Já Programou" na aba Perfis), 
                                indicando que eles enxergam a tecnologia como uma ferramenta transversal, capaz de se conectar com áreas de Humanas através de jogos, narrativas digitais e preservação histórica.
                            </p>

                            <h3>4. Conclusão</h3>
                            <p>
                                O evento foi eficaz em atrair estudantes com perfil compatível. Para futuras edições, recomenda-se ações específicas para desmistificar a área de exatas para o público feminino 
                                e demonstrar as aplicações criativas da computação (jogos, design) para atrair alunos com perfil mais artístico/humanas.
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
import BarChart from './BarChart.vue'
import PieChart from './PieChart.vue'
import Navbar from './Navbar.vue'

const stats = ref(null)
const loading = ref(true)
const error = ref(null)
const currentTab = ref('geral')
const selectedGender = ref('')

const tabs = [
    { id: 'geral', label: 'Visão Geral' },
    { id: 'escolaridade', label: 'Escolaridade' },
    { id: 'pretensao', label: 'Pretensão' },
    { id: 'materias', label: 'Matérias Escolhidas' },
    { id: 'tecnologia', label: 'Perfil Tecnológico' },
    { id: 'perfis', label: 'Perfis' },
    { id: 'analise', label: 'Análise Interpretativa' }
]

const fetchData = async () => {
    loading.value = true
    try {
        let url = 'https://a3-la3z.onrender.com/estatisticas'
        if (selectedGender.value) {
            url += `?genero=${selectedGender.value}`
        }
        const response = await axios.get(url)
        stats.value = response.data
    } catch (err) {
        error.value = 'Erro ao carregar dados: ' + err.message
        console.error(err)
    } finally {
        loading.value = false
    }
}

const formatProfileTitle = (key) => {
    const map = {
        'homens_2ano': 'Homens - 2º Ano',
        'homens_3ano': 'Homens - 3º Ano',
        'mulheres_2ano': 'Mulheres - 2º Ano',
        'mulheres_3ano': 'Mulheres - 3º Ano',
        'nb_2ano': 'Não Binário - 2º Ano',
        'nb_3ano': 'Não Binário - 3º Ano'
    }
    return map[key] || key
}

const profileKeys = [
    'homens_2ano', 'homens_3ano', 
    'mulheres_2ano', 'mulheres_3ano', 
    'nb_2ano', 'nb_3ano'
]

onMounted(() => {
    fetchData()
})

const generateChartData = (dataObj, label, colors) => {
    if (!dataObj) return null
    const labels = Object.keys(dataObj)
    const data = Object.values(dataObj)

    return {
        labels,
        datasets: [
            {
                label: label,
                backgroundColor: colors || ['#41B883', '#E46651', '#00D8FF', '#DD1B16', '#f87979'],
                data
            }
        ]
    }
}

const genderData = computed(() => {
    if (!stats.value || !stats.value.demografia) return null
    
    const rawData = stats.value.demografia.genero
    const labels = Object.keys(rawData)
    const data = Object.values(rawData)
    
    // Cores fixas para Homem e Mulher se possível, senão usa padrão
    const backgroundColors = labels.map(label => {
        const l = label.toLowerCase()
        if (l.includes('homem')) return '#36A2EB' // Azul
        if (l.includes('mulher')) return '#FF6384' // Rosa
        if (l.includes('não binário') || l.includes('nao binario')) return '#9966FF' // Roxo
        if (l.includes('prefiro não dizer')) return '#FF9F40' // Laranja
        return '#C9CBCF' // Cinza para outros
    })

    return {
        labels,
        datasets: [
            {
                label: 'Gênero',
                backgroundColor: backgroundColors,
                data
            }
        ]
    }
})


const schoolYearData = computed(() => {
    if (!stats.value || !stats.value.demografia) return null
    return generateChartData(stats.value.demografia.ano_escolar, 'Ano Escolar', ['#4BC0C0', '#9966FF', '#FF9F40'])
})

const universityData = computed(() => {
    if (!stats.value || !stats.value.demografia) return null
    return generateChartData(stats.value.demografia.universidade_pretendida, 'Universidade', ['#FF6384', '#36A2EB'])
})

const generalEvalData = computed(() => {
    if (!stats.value || !stats.value.avaliacoes) return null
    return generateChartData(stats.value.avaliacoes.experiencia_geral, 'Avaliação Geral')
})

// Novos gráficos para Matérias
const techInterestData = computed(() => {
    if (!stats.value || !stats.value.interesses_areas) return null
    return generateChartData(stats.value.interesses_areas.tecnologia, 'Interesse em Tecnologia')
})

const challengesInterestData = computed(() => {
    if (!stats.value || !stats.value.interesses_areas) return null
    return generateChartData(stats.value.interesses_areas.desafios, 'Interesse em Desafios')
})

const mathInterestData = computed(() => {
    if (!stats.value || !stats.value.interesses_areas) return null
    return generateChartData(stats.value.interesses_areas.matematica, 'Interesse em Matemática')
})

const portugueseInterestData = computed(() => {
    if (!stats.value || !stats.value.interesses_areas) return null
    return generateChartData(stats.value.interesses_areas.portugues, 'Interesse em Português')
})

const favoriteSubjectData = computed(() => {
    if (!stats.value || !stats.value.interesses_areas) return null
    return generateChartData(stats.value.interesses_areas.materia_preferida, 'Matéria Preferida')
})

// Novos gráficos para Perfil Tecnológico
const shiftPreferenceData = computed(() => {
    if (!stats.value || !stats.value.perfil_tecnologico) return null
    return generateChartData(stats.value.perfil_tecnologico.turno_preferencia, 'Turno de Preferência')
})

const programmingContactData = computed(() => {
    if (!stats.value || !stats.value.perfil_tecnologico) return null
    return generateChartData(stats.value.perfil_tecnologico.contato_programacao, 'Contato com Programação')
})

const likesGamesData = computed(() => {
    if (!stats.value || !stats.value.perfil_tecnologico) return null
    return generateChartData(stats.value.perfil_tecnologico.gosta_jogos, 'Gosta de Jogos')
})

const devicesData = computed(() => {
    if (!stats.value || !stats.value.perfil_tecnologico || !stats.value.perfil_tecnologico.dispositivos) return null
    
    const dispositivos = stats.value.perfil_tecnologico.dispositivos
    const labels = ['Videogame', 'Computador', 'Internet', 'Celular', 'Internet Celular']
    const data = [
        dispositivos.videogame?.['Sim'] || 0,
        dispositivos.computador?.['Sim'] || 0,
        dispositivos.internet?.['Sim'] || 0,
        dispositivos.celular?.['Sim'] || 0,
        dispositivos.internet_celular?.['Sim'] || 0
    ]

    return {
        labels,
        datasets: [
            {
                label: 'Possui Dispositivo (Sim)',
                backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0', '#9966FF'],
                data
            }
        ]
    }
})

</script>

<style scoped>
.dashboard-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

.loading,
.error,
.no-data {
    font-size: 1.5rem;
    margin-top: 50px;
    text-align: center;
}

.error {
    color: #ff6b6b;
}

.summary-cards {
    display: flex;
    justify-content: center;
    margin-bottom: 40px;
}

.card {
    background: #333;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    text-align: center;
    min-width: 200px;
}

.big-number {
    font-size: 3rem;
    font-weight: bold;
    margin: 10px 0 0;
    color: #41B883;
}

.charts-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
}

.full-width {
    grid-column: 1 / -1;
}

.chart-container {
    background: #2a2a2a;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    height: 450px;
    position: relative;
    display: flex;
    flex-direction: column;
}

.chart-wrapper {
    flex: 1;
    position: relative;
    min-height: 0;
    width: 100%;
}

h3 {
    margin-bottom: 15px;
    color: #fff;
    font-size: 1.1rem;
    text-align: center;
}

.filters-container {
    margin-bottom: 20px;
    display: flex;
    align-items: center;
    gap: 10px;
    background: #2a2a2a;
    padding: 10px 20px;
    border-radius: 8px;
}

.filters-container label {
    color: #fff;
    font-weight: bold;
}

.filters-container select {
    padding: 5px 10px;
    border-radius: 4px;
    border: 1px solid #444;
    background: #333;
    color: #fff;
}

.profiles-section {
    display: flex;
    flex-direction: column;
    gap: 30px;
}

.section-title {
    color: #41B883;
    border-bottom: 2px solid #41B883;
    padding-bottom: 10px;
    margin-bottom: 20px;
}

.profiles-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
}

.profile-card {
    background: #2a2a2a;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    border-left: 4px solid #41B883;
}

.profile-card h4 {
    color: #fff;
    margin-bottom: 15px;
    font-size: 1.2rem;
}

.profile-card p {
    margin: 8px 0;
    color: #ccc;
}

.profile-card strong {
    color: #fff;
}

.no-profile-data {
    color: #888;
    font-style: italic;
}

.analysis-section {
    max-width: 800px;
    margin: 0 auto;
}

.analysis-card {
    background: #2a2a2a;
    padding: 40px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.analysis-text {
    color: #ddd;
    line-height: 1.6;
    font-size: 1.1rem;
}

.analysis-text h3 {
    color: #41B883;
    margin-top: 30px;
    margin-bottom: 15px;
    text-align: left;
    border-bottom: 1px solid #444;
    padding-bottom: 5px;
}

.analysis-text p {
    margin-bottom: 15px;
    text-align: justify;
}

.small-text {
    font-size: 1rem;
    color: #888;
    margin-top: -5px;
}
</style>
