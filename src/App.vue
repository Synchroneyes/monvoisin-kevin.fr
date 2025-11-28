<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { 
  Github, Linkedin, Mail, ArrowRight, X,
  Cloud, Code2, GitBranch, Terminal, Gauge, Server, 
  Boxes, Package, Workflow, Database, Shield, Cpu,
  Network, Layers, FileCode, Zap, Settings, Container
} from 'lucide-vue-next'

const availableIcons: Record<string, any> = {
  'cloud': Cloud,
  'code': Code2,
  'gitbranch': GitBranch,
  'terminal': Terminal,
  'gauge': Gauge,
  'server': Server,
  'boxes': Boxes,
  'package': Package,
  'workflow': Workflow,
  'database': Database,
  'shield': Shield,
  'cpu': Cpu,
  'network': Network,
  'layers': Layers,
  'filecode': FileCode,
  'zap': Zap,
  'settings': Settings,
  'container': Container
}

const iconMap: Record<string, any> = {
  'Cloud': Cloud,
  'DevOps': Code2,
  'CI/CD': GitBranch,
  'Monitoring': Gauge,
  'Scripting': Terminal,
  'Infrastructure as Code': Layers
}

const colorMap: Record<string, string> = {
  'Cloud': 'from-blue-500 to-cyan-500',
  'DevOps': 'from-emerald-500 to-teal-500',
  'CI/CD': 'from-purple-500 to-pink-500',
  'Monitoring': 'from-orange-500 to-red-500',
  'Scripting': 'from-yellow-500 to-orange-500',
  'Infrastructure as Code': 'from-indigo-500 to-purple-500'
}

interface Skill {
  name: string
  items: string[]
  icon: any
  color: string
}

interface Experience {
  role: string
  company: string
  period: string
  description: string
}

interface Article {
  title: string
  url: string
  date: string
  description: string
  tags: string[]
}

interface Social {
  name: string
  icon: any
  url: string
}

const skills = ref<Skill[]>([])
const experiences = ref<Experience[]>([])
const certifications = ref<string[]>([])
const articles = ref<Article[]>([])
const socials = ref<Social[]>([])
const rotatingTexts = ref<string[]>([])
const heroDescription = ref('')
const loading = ref(true)
const modalOpen = ref(false)
const selectedExperience = ref<Experience | null>(null)

const scrollToContact = () => {
  document.getElementById('contact')?.scrollIntoView({ behavior: 'smooth' })
}

const openModal = (exp: Experience) => {
  selectedExperience.value = exp
  modalOpen.value = true
  document.body.style.overflow = 'hidden'
}

const closeModal = () => {
  modalOpen.value = false
  selectedExperience.value = null
  document.body.style.overflow = 'auto'
}

const truncateText = (text: string, maxLength: number) => {
  if (text.length <= maxLength) return text
  return text.substring(0, maxLength) + '...'
}

// Load data from markdown files
const loadSkills = async () => {
  try {
    const response = await fetch('/content/skills.md')
    const text = await response.text()
    const sections = text.split('##').filter(s => s.trim())
    
    skills.value = sections.map(section => {
      const lines = section.trim().split('\n')
      const firstLine = lines[0]?.trim() || ''
      
      // Parse title and optional icon: ## Cloud [icon:cloud]
      const iconMatch = firstLine.match(/\[icon:(.*?)\]/)
      const name = firstLine.replace(/\[icon:.*?\]/, '').trim()
      const iconName = iconMatch?.[1]?.toLowerCase() || null
      
      const content = lines.slice(1).filter(l => l.trim()).join('\n')
      
      // Parse markdown: bold, code, links
      const items = content.split(',').map(item => {
        return item.trim()
          .replace(/\*\*(.*?)\*\*/g, '<strong class="text-white">$1</strong>')
          .replace(/`(.*?)`/g, '<code class="text-emerald-400 bg-[#1a1a1a] px-1 rounded">$1</code>')
          .replace(/\[(.*?)\]\((.*?)\)/g, '<a href="$2" target="_blank" class="text-emerald-400 hover:underline">$1</a>')
      })
      
      // Select icon: custom > default map > fallback
      let selectedIcon = Code2
      if (iconName && availableIcons[iconName]) {
        selectedIcon = availableIcons[iconName]
      } else if (iconMap[name]) {
        selectedIcon = iconMap[name]
      }
      
      return {
        name,
        items,
        icon: selectedIcon,
        color: colorMap[name] || 'from-gray-500 to-gray-600'
      }
    })
  } catch (error) {
    console.error('Error loading skills:', error)
    // Fallback data
    skills.value = [
      { icon: Cloud, name: 'Cloud', items: ['AWS', 'Azure', 'GCP'], color: 'from-blue-500 to-cyan-500' },
      { icon: Code2, name: 'DevOps', items: ['Docker', 'Kubernetes', 'Terraform'], color: 'from-emerald-500 to-teal-500' },
      { icon: GitBranch, name: 'CI/CD', items: ['GitLab CI', 'GitHub Actions', 'Jenkins'], color: 'from-purple-500 to-pink-500' }
    ]
  }
}

const loadExperiences = async () => {
  try {
    const response = await fetch('/content/experience.md')
    const text = await response.text()
    const blocks = text.split('---').filter(s => s.trim())
    
    experiences.value = blocks.map(block => {
      const lines = block.trim().split('\n')
      const exp: any = {}
      let currentKey = ''
      
      lines.forEach(line => {
        const colonIndex = line.indexOf(':')
        if (colonIndex > 0 && ['role', 'company', 'period', 'description'].includes(line.substring(0, colonIndex).trim())) {
          currentKey = line.substring(0, colonIndex).trim()
          exp[currentKey] = line.substring(colonIndex + 1).trim()
        } else if (currentKey) {
          // Continue multiline for current key (preserve empty lines)
          exp[currentKey] += '\n' + line.trim()
        }
      })
      return exp as Experience
    })
  } catch (error) {
    console.error('Error loading experiences:', error)
    // Fallback data
    experiences.value = [
      {
        role: 'DevOps Engineer Senior',
        company: 'Entreprise Tech Inc.',
        period: '2022 - Présent',
        description: 'Migration cloud AWS, CI/CD avec GitLab et ArgoCD, orchestration de +50 microservices Kubernetes.'
      },
      {
        role: 'DevOps Engineer',
        company: 'StartupXYZ',
        period: '2020 - 2021',
        description: 'Infrastructure Azure from scratch, automatisation Jenkins/Ansible, monitoring ELK Stack.'
      }
    ]
  }
}

const loadCertifications = async () => {
  try {
    const response = await fetch('/content/certifications.md')
    const text = await response.text()
    certifications.value = text.split('\n').filter(line => line.trim())
  } catch (error) {
    console.error('Error loading certifications:', error)
    // Fallback data
    certifications.value = [
      'AWS Solutions Architect Professional',
      'Certified Kubernetes Administrator',
      'HashiCorp Terraform Associate'
    ]
  }
}

const loadArticles = async () => {
  try {
    const response = await fetch('/content/articles.md')
    const text = await response.text()
    const blocks = text.split('---').filter(s => s.trim())
    
    articles.value = blocks.map(block => {
      const lines = block.trim().split('\n')
      const article: any = {}
      lines.forEach(line => {
        const [key, ...values] = line.split(':')
        if (key && values.length) {
          const value = values.join(':').trim()
          if (key.trim() === 'tags') {
            article.tags = value.split(',').map(t => t.trim())
          } else {
            article[key.trim()] = value
          }
        }
      })
      return article as Article
    })
  } catch (error) {
    console.error('Error loading articles:', error)
    articles.value = []
  }
}

const loadSocials = async () => {
  try {
    const response = await fetch('/content/socials.md')
    const text = await response.text()
    const sections = text.split('##').filter(s => s.trim())
    
    socials.value = sections.map(section => {
      const lines = section.trim().split('\n')
      const name = lines[0]?.trim() || ''
      let iconName = ''
      let url = ''
      
      lines.forEach(line => {
        if (line.startsWith('icon:')) {
          iconName = line.replace('icon:', '').trim().toLowerCase()
        } else if (line.startsWith('url:')) {
          url = line.replace('url:', '').trim()
        }
      })
      
      const iconMap: Record<string, any> = {
        'github': Github,
        'linkedin': Linkedin,
        'mail': Mail
      }
      
      return {
        name,
        icon: iconMap[iconName] || Mail,
        url
      }
    }).filter(s => s.name && s.url)
  } catch (error) {
    console.error('Error loading socials:', error)
    socials.value = [
      { name: 'GitHub', icon: Github, url: 'https://github.com/kevinmonvoisin' },
      { name: 'LinkedIn', icon: Linkedin, url: 'https://linkedin.com/in/kevin-monvoisin' },
      { name: 'Email', icon: Mail, url: 'mailto:kevin@monvoisin-kevin.fr' }
    ]
  }
}

const loadHero = async () => {
  try {
    const response = await fetch('/content/hero.md')
    const text = await response.text()
    
    const sections = text.split('# Description')
    if (sections.length > 0 && sections[0]) {
      const rotatingSection = sections[0].replace('# Textes rotatifs', '').trim()
      rotatingTexts.value = rotatingSection.split('\n').filter(line => line.trim())
    }
    if (sections.length > 1 && sections[1]) {
      heroDescription.value = sections[1].trim()
    }
    
    // Fallback data
    if (rotatingTexts.value.length === 0) {
      rotatingTexts.value = ['DevOps Engineer', 'Cloud Architect', 'Infrastructure Expert']
    }
    if (!heroDescription.value) {
      heroDescription.value = 'Passionné par l\'infrastructure cloud et l\'automatisation.'
    }
  } catch (error) {
    console.error('Error loading hero:', error)
    rotatingTexts.value = ['DevOps Engineer', 'Cloud Architect', 'Infrastructure Expert']
    heroDescription.value = 'Passionné par l\'infrastructure cloud et l\'automatisation.'
  }
}

// Rotating text animation
const currentRotatingText = ref('')
let rotatingInterval: number

onMounted(async () => {
  await Promise.all([loadSkills(), loadExperiences(), loadCertifications(), loadArticles(), loadHero(), loadSocials()])
  loading.value = false
  
  if (rotatingTexts.value.length > 0) {
    currentRotatingText.value = rotatingTexts.value[0] || ''
    let index = 0
    rotatingInterval = setInterval(() => {
      index = (index + 1) % rotatingTexts.value.length
      currentRotatingText.value = rotatingTexts.value[index] || ''
    }, 3000) as unknown as number
  }
})

onUnmounted(() => {
  if (rotatingInterval) {
    clearInterval(rotatingInterval)
  }
})
</script>

<template>
  <div class="min-h-screen bg-[#0a0a0a] text-[#e5e5e5]">
    <!-- Navigation -->
    <nav class="fixed top-0 w-full bg-[#0a0a0a]/80 backdrop-blur-sm border-b border-[#1a1a1a] z-50">
      <div class="max-w-6xl mx-auto px-6 py-4 flex justify-between items-center">
        <div class="text-lg font-semibold">KM</div>
        <div class="flex gap-6">
          <a 
            v-for="social in socials" 
            :key="social.name"
            :href="social.url" 
            target="_blank" 
            rel="noopener noreferrer"
            class="text-[#888] hover:text-white transition-colors"
          >
            <component :is="social.icon" class="h-5 w-5" />
          </a>
        </div>
      </div>
    </nav>

    <!-- Hero Section -->
    <section class="pt-32 pb-20 px-6">
      <div class="max-w-4xl mx-auto">
        <div class="flex flex-col md:flex-row md:items-start md:justify-between gap-8">
          <div class="flex-1">
            <div class="mb-4">
              <span class="text-[#888] text-sm">Bonjour, je suis</span>
            </div>
            <h1 class="text-6xl md:text-7xl font-bold mb-6 tracking-tight">
              <span class="bg-gradient-to-r from-pink-400 via-orange-400 to-red-500 bg-clip-text text-transparent">
                Kevin Monvoisin
              </span>
            </h1>
            <div class="h-20 md:h-24 mb-8">
              <p class="text-2xl md:text-3xl text-[#888] font-light">
                <Transition name="slide-fade" mode="out-in">
                  <span :key="currentRotatingText" class="inline-block">
                    <span class="text-orange-400">{{ currentRotatingText }}</span>
                  </span>
                </Transition>
              </p>
            </div>
            <p class="text-lg text-[#666] mb-12 max-w-2xl leading-relaxed">
              {{ heroDescription }}
            </p>
            <button 
              @click="scrollToContact"
              class="inline-flex items-center gap-2 bg-gradient-to-r from-pink-500 via-orange-500 to-red-500 text-white px-6 py-3 rounded-lg hover:from-pink-600 hover:via-orange-600 hover:to-red-600 transition-all font-medium shadow-lg shadow-orange-500/20"
            >
              Me contacter
              <ArrowRight class="h-4 w-4" />
            </button>
          </div>
          
          <!-- Profile Picture -->
          <div class="relative group">
            <div class="absolute inset-0 bg-gradient-to-r from-pink-400 via-orange-400 to-red-500 rounded-full blur-xl opacity-75 group-hover:opacity-100 transition-opacity"></div>
            <div class="relative p-1 bg-gradient-to-r from-pink-400 via-orange-400 to-red-500 rounded-full">
              <img 
                src="/profile.jpg" 
                alt="Kevin Monvoisin" 
                class="w-48 h-48 md:w-64 md:h-64 rounded-full object-cover bg-[#0a0a0a]"
              />
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Articles Section -->
    <section v-if="articles.length > 0" class="py-20 px-6 border-t border-[#1a1a1a]">
      <div class="max-w-4xl mx-auto">
        <h2 class="text-3xl font-bold mb-12">Articles Medium</h2>
        <div v-if="loading" class="text-center text-[#888]">Chargement...</div>
        <div v-else class="grid md:grid-cols-2 gap-6">
          <a 
            v-for="article in articles" 
            :key="article.url"
            :href="article.url"
            target="_blank"
            rel="noopener noreferrer"
            class="border border-[#1a1a1a] rounded-lg p-6 hover:border-orange-500/30 hover:bg-[#0f0f0f] transition-all group block"
          >
            <div class="flex items-start justify-between mb-3">
              <h3 class="text-lg font-semibold group-hover:text-orange-400 transition-colors pr-4">{{ article.title }}</h3>
              <ArrowRight class="h-5 w-5 text-[#666] group-hover:text-orange-400 group-hover:translate-x-1 transition-all flex-shrink-0" />
            </div>
            <p class="text-sm text-[#888] mb-4">{{ article.date }}</p>
            <p class="text-[#666] text-sm mb-4 line-clamp-2">{{ article.description }}</p>
            <div class="flex flex-wrap gap-2">
              <span 
                v-for="tag in article.tags" 
                :key="tag"
                class="text-xs px-2 py-1 rounded bg-orange-500/10 text-orange-400 border border-orange-500/20"
              >
                {{ tag }}
              </span>
            </div>
          </a>
        </div>
      </div>
    </section>

    <!-- Skills Section -->
    <section class="py-20 px-6 border-t border-[#1a1a1a]">
      <div class="max-w-4xl mx-auto">
        <h2 class="text-3xl font-bold mb-12">Compétences</h2>
        <div v-if="loading" class="text-center text-[#888]">Chargement...</div>
        <div v-else class="grid md:grid-cols-3 gap-8">
          <div v-for="skill in skills" :key="skill.name" class="group relative overflow-hidden rounded-xl">
            <div :class="['absolute inset-0 bg-gradient-to-br opacity-0 group-hover:opacity-10 transition-opacity', skill.color]"></div>
            <div class="relative border border-[#1a1a1a] rounded-xl p-6 group-hover:border-[#2a2a2a] transition-colors h-full">
              <div :class="['w-12 h-12 rounded-lg bg-gradient-to-br flex items-center justify-center mb-4', skill.color]">
                <component :is="skill.icon" class="h-6 w-6 text-white" />
              </div>
              <h3 class="text-xl font-semibold mb-3">{{ skill.name }}</h3>
              <ul class="space-y-2">
                <li v-for="item in skill.items" :key="item" class="text-[#888] text-sm flex items-start gap-2">
                  <span class="text-orange-400 mt-1">•</span>
                  <span v-html="item"></span>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Experience Section -->
    <section class="py-20 px-6 border-t border-[#1a1a1a]">
      <div class="max-w-4xl mx-auto">
        <h2 class="text-3xl font-bold mb-12">Expérience</h2>
        <div v-if="loading" class="text-center text-[#888]">Chargement...</div>
        <div v-else class="relative">
          <!-- Timeline line -->
          <div class="absolute left-8 top-0 bottom-0 w-px bg-gradient-to-b from-pink-500 via-orange-500 to-red-500"></div>
          
          <div class="space-y-0">
            <div v-for="exp in experiences" :key="exp.role" class="relative group pb-8">
              <!-- Timeline dot -->
              <div class="absolute left-8 top-6 w-4 h-4 -ml-2 rounded-full bg-gradient-to-br from-pink-500 via-orange-500 to-red-500 ring-4 ring-[#0a0a0a] group-hover:scale-125 transition-transform z-10"></div>
              
              <!-- Content card -->
              <div class="ml-20">
                <!-- Date period above card -->
                <div class="text-xs font-mono text-orange-400 mb-2 flex items-center gap-2">
                  <span class="bg-orange-500/10 px-2 py-1 rounded border border-orange-500/20">{{ exp.period }}</span>
                </div>
                
                <div 
                  @click="openModal(exp)"
                  class="border border-[#1a1a1a] rounded-lg p-6 bg-[#0f0f0f] group-hover:border-orange-500/30 group-hover:bg-[#121212] transition-all cursor-pointer"
                >
                  <h3 class="text-xl font-semibold group-hover:text-orange-400 transition-colors mb-2">{{ exp.role }}</h3>
                  <p class="text-[#888] font-medium mb-3">{{ exp.company }}</p>
                  <p class="text-[#666] text-sm whitespace-pre-line">{{ truncateText(exp.description, 200) }}</p>
                  <p class="text-orange-400 text-xs mt-3 opacity-0 group-hover:opacity-100 transition-opacity">Cliquer pour voir plus →</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Certifications Section -->
    <section class="py-20 px-6 border-t border-[#1a1a1a]">
      <div class="max-w-4xl mx-auto">
        <h2 class="text-3xl font-bold mb-12">Certifications</h2>
        <div v-if="loading" class="text-center text-[#888]">Chargement...</div>
        <div v-else class="grid md:grid-cols-2 gap-4">
          <div 
            v-for="cert in certifications" 
            :key="cert"
            class="border border-[#1a1a1a] rounded-lg p-6 hover:border-pink-500/30 hover:bg-pink-500/5 transition-all group"
          >
            <div class="flex items-center gap-3">
              <div class="w-2 h-2 rounded-full bg-pink-500"></div>
              <p class="font-medium group-hover:text-pink-400 transition-colors">{{ cert }}</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Contact Section -->
    <section id="contact" class="py-20 px-6 border-t border-[#1a1a1a]">
      <div class="max-w-4xl mx-auto text-center">
        <h2 class="text-3xl font-bold mb-6">
          Connectons <span class="bg-gradient-to-r from-pink-400 via-orange-400 to-red-400 bg-clip-text text-transparent">nous</span>
        </h2>
        <p class="text-[#888] mb-8 max-w-2xl mx-auto">
          Une question, un projet ou simplement envie d'échanger ? 
          Je serais ravi de discuter avec vous.
        </p>
        <a 
          href="https://linkedin.com/in/kevin-monvoisin"
          target="_blank"
          rel="noopener noreferrer"
          class="inline-flex items-center gap-2 bg-gradient-to-r from-pink-500 via-orange-500 to-red-500 text-white px-8 py-4 rounded-lg hover:from-pink-600 hover:via-orange-600 hover:to-red-600 transition-all font-medium text-lg shadow-lg shadow-orange-500/20"
        >
          <Linkedin class="h-5 w-5" />
          LinkedIn
        </a>
      </div>
    </section>

    <!-- Footer -->
    <footer class="border-t border-[#1a1a1a] py-8">
      <div class="max-w-4xl mx-auto px-6 text-center text-sm text-[#666]">
        <p>© 2025 Kevin Monvoisin</p>
      </div>
    </footer>

    <!-- Modal -->
    <Transition name="modal">
      <div 
        v-if="modalOpen" 
        @click="closeModal"
        class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/80 backdrop-blur-sm"
      >
        <div 
          @click.stop
          class="relative bg-[#0f0f0f] border border-[#1a1a1a] rounded-xl max-w-3xl w-full max-h-[80vh] overflow-y-auto"
        >
          <!-- Close button -->
          <button 
            @click="closeModal"
            class="sticky top-4 float-right m-4 p-2 rounded-lg bg-[#1a1a1a] hover:bg-[#2a2a2a] transition-colors z-10"
          >
            <X class="h-5 w-5 text-[#888]" />
          </button>
          
          <!-- Modal content -->
          <div class="p-8" v-if="selectedExperience">
            <div class="mb-4">
              <span class="text-xs font-mono text-orange-400 bg-orange-500/10 px-3 py-1 rounded border border-orange-500/20">
                {{ selectedExperience.period }}
              </span>
            </div>
            <h2 class="text-3xl font-bold mb-2 bg-gradient-to-r from-pink-400 via-orange-400 to-red-500 bg-clip-text text-transparent">
              {{ selectedExperience.role }}
            </h2>
            <p class="text-xl text-[#888] font-medium mb-6">{{ selectedExperience.company }}</p>
            <div class="text-[#ddd] whitespace-pre-line leading-relaxed">
              {{ selectedExperience.description }}
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style>
html {
  scroll-behavior: smooth;
}

.slide-fade-enter-active {
  transition: all 0.5s ease;
}

.slide-fade-leave-active {
  transition: all 0.3s ease;
}

.slide-fade-enter-from {
  transform: translateY(20px);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-active > div,
.modal-leave-active > div {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from > div,
.modal-leave-to > div {
  transform: scale(0.9);
  opacity: 0;
}
</style>
