<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

useHead({
  title: 'Personal Portfolio',
  link: [
    {
      rel: 'stylesheet',
      href: 'https://fonts.googleapis.com/css2?family=Bodoni+Moda:ital,opsz,wght@0,6..96,400;0,6..96,500;0,6..96,600;0,6..96,700;1,6..96,400&family=Manrope:wght@400;500;600;700;800&display=swap'
    }
  ]
})

const skillItems = ref([
  { id: 'vue', name: 'Vue', meta: '80%', img: '/img/skills/vue.png', lab: false },
  { id: 'nuxt', name: 'Nuxt', meta: '85%', img: '/img/skills/nuxt.png', lab: false },
  { id: 'tailwind', name: 'Tailwind CSS', meta: '70%', img: '/img/skills/tailwind.png', lab: false },
  { id: 'laravel', name: 'Laravel', meta: '95%', img: '/img/skills/laravel.png', lab: false },
  { id: 'docker', name: 'Docker', meta: '85%', img: '/img/skills/docker.png', lab: false },
])

const ORBIT_DURATION = 48 

const orbitItems = computed(() => {
  const n = skillItems.value.length
  return skillItems.value.map((item, i) => {
    const angle = (360 / n) * i
    const delay = -(angle / 360) * ORBIT_DURATION
    const enterDelay = 0.12 + i * 0.14
    const initials = item.name
      .replace(/[^A-Za-z0-9]/g, ' ')
      .trim()
      .split(' ')
      .map((w) => w[0])
      .slice(0, 2)
      .join('')
      .toUpperCase()
    return { ...item, angle, delay, enterDelay, initials }
  })
})

function handleImgError(event) {
  event.target.style.display = 'none'
  const fallback = event.target.nextElementSibling
  if (fallback) fallback.style.display = 'flex'
}

const swingRef = ref(null)
const isDragging = ref(false)
const isSettling = ref(false)
const hasEntered = ref(false)
const dragAngle = ref(0)
let settleTimeout = null
let entranceTimeout = null
let pointerStart = { x: 0, y: 0 }
let hasMoved = false
const MOVE_THRESHOLD = 6

function getAnchor(el) {
  const rect = el.getBoundingClientRect()
  return { x: rect.left + rect.width / 2, y: rect.top }
}

function updateAngle(clientX, clientY) {
  if (!swingRef.value) return
  const anchor = getAnchor(swingRef.value)
  const dx = clientX - anchor.x
  const dy = Math.max(clientY - anchor.y, 1)
  let angle = -Math.atan2(dx, dy) * (180 / Math.PI)
  angle = Math.max(-75, Math.min(75, angle))
  dragAngle.value = angle
}

function onPointerDown(e) {
  pointerStart = { x: e.clientX, y: e.clientY }
  hasMoved = false
  isSettling.value = false
  clearTimeout(settleTimeout)
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', onPointerUp)
}

function onPointerMove(e) {
  const dist = Math.hypot(e.clientX - pointerStart.x, e.clientY - pointerStart.y)
  if (!hasMoved && dist > MOVE_THRESHOLD) {
    hasMoved = true
    isDragging.value = true
  }
  if (isDragging.value) {
    updateAngle(e.clientX, e.clientY)
  }
}

function onPointerUp() {
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)

  if (hasMoved) {
    isDragging.value = false
    isSettling.value = true
    settleTimeout = setTimeout(() => {
      isSettling.value = false
      dragAngle.value = 0
    }, 950)
  }
}

const swingStyle = computed(() => {
  if (isDragging.value) {
    return { transform: `rotate(${dragAngle.value}deg)`, transition: 'none', animation: 'none' }
  }
  if (isSettling.value) {
    return {
      transform: 'rotate(0deg)',
      transition: 'transform 0.95s cubic-bezier(0.22, 1.35, 0.36, 1)',
      animation: 'none'
    }
  }
  return {}
})

onMounted(() => {
  entranceTimeout = setTimeout(() => {
    hasEntered.value = true
  }, 1150)
})

onUnmounted(() => {
  clearTimeout(settleTimeout)
  clearTimeout(entranceTimeout)
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
})

const educations = ref([
  { year: '2024 - Now', role: 'SMKN 1 Gunungputri ', place: 'Rekayasa Perangkat Lunak' },
])

const experiences = ref([
  { year: 'Jun - Sep 2026', role: 'IT Internship', place: 'PT. Van Aroma' },
  { year: 'May 2026', role: 'Selected Exhibitor LCGN', place: 'PT.Clevio' },
  { year: 'Feb 2025', role: 'Semifinalist LCGN', place: 'PT.Clevio' },
  { year: 'Jan - Apr 2025', role: 'Dicoding FEBE boothcamp scholarship', place: 'Codingcamp by DBS Foundation' },
])

const activeStuffTab = ref('projects')

const stuffTabs = [
  { id: 'projects', label: 'Project' },
  { id: 'certificates', label: 'Sertifikat' },
  { id: 'techstack', label: 'Tech Stack' },
]

const stuffProjects = ref([
  {
    title: 'foto',
    tags: 'HTML . CSS . Javascript',
    desc: 'Project untuk kompetisi LCGN pertama, dibangun pakai HTML, CSS, dan JavaScript.',
    img: '/img/projects/lcgn-1.png',
  },
  {
    title: 'foto',
    tags: 'Laravel . Xampp . Tailwind',
    desc: 'Tugas kelompok berbasis Laravel dengan styling Tailwind CSS.',
    img: '/img/projects/tugas-kelompok-1.png',
  },
  {
    title: 'foto',
    tags: 'HTML . CSS . Javascript',
    desc: 'Iterasi kedua project LCGN dengan perbaikan UI dan fitur.',
    img: '/img/projects/lcgn-2.png',
  },
  {
    title: 'foto',
    tags: 'Laravel . Docker',
    desc: 'Project individu menggunakan Laravel yang di-deploy dengan Docker.',
    img: '/img/projects/tugas-individu.png',
  },
  {
    title: 'foto',
    tags: 'Laravel . Docker',
    desc: 'Sistem manajemen kunjungan tamu untuk PT. Van Aroma.',
    img: '/img/projects/van-aroma-visitor.png',
  },
  {
    title: 'foto',
    tags: 'Laravel . Docker . Nuxt',
    desc: 'Sistem keamanan internal untuk kebutuhan operasional Van Aroma.',
    img: '/img/projects/van-aroma-security.png',
  },
  {
    title: 'foto',
    tags: 'Laravel . Docker . Nuxt . Tailwind',
    desc: 'Portal lowongan kerja internal untuk PT. Van Aroma.',
    img: '/img/projects/van-aroma-job-portal.png',
  },
])

const stuffCertificates = ref([
  { title: '', issuer: 'Komdigi', year: '2026', img: '/img/sertif/digi.jpg' },
  { title: '', issuer: 'Clevio Camp', year: '2026', img: '/img/sertif/lcgn2.jpg' },
  { title: '', issuer: 'Dicoding', year: '2025', img: '/img/sertif/coding1.png' },
  { title: '', issuer: 'Clevio Camp', year: '2025', img: '/img/sertif/lcgn1.png' },
  { title: '', issuer: 'Dicoding', year: '2025', img: '/img/sertif/coding2.png' },
  { title: '', issuer: 'Dicoding', year: '2025', img: '/img/sertif/coding3.png' },
  { title: '', issuer: 'Coursera', year: '2025', img: '/img/sertif/coursera.jpg' },
  { title: '', issuer: 'Dicoding', year: '2025', img: '/img/sertif/coding4.png' },
  { title: '', issuer: 'Dicoding', year: '2025', img: '/img/sertif/coding5.png' },

])

const lightboxImage = ref(null)
const lightboxAlt = ref('')

function openLightbox(img, alt) {
  if (!img) return
  lightboxImage.value = img
  lightboxAlt.value = alt || ''
}

function closeLightbox() {
  lightboxImage.value = null
  lightboxAlt.value = ''
}

function onLightboxKeydown(e) {
  if (e.key === 'Escape') closeLightbox()
}

onMounted(() => {
  window.addEventListener('keydown', onLightboxKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', onLightboxKeydown)
})

const stuffTechStack = ref([
  { name: 'i will add it ASAP', category: 'i just think about this small potato stuff' },
])

const navItems = [
  { id: 'home', label: 'Home' },
  { id: 'about', label: 'About' },
  { id: 'skills', label: 'Skills' },
  { id: 'potatoes', label: 'potatoes' },
  { id: 'contact', label: 'Contact' },
]

const mobileMenuOpen = ref(false)
const navScrolled = ref(false)
const activeSection = ref('home')
let sectionObserver = null

function scrollToSection(id) {
  const el = document.getElementById(id)
  if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' })
  mobileMenuOpen.value = false
}

function handleNavScroll() {
  navScrolled.value = window.scrollY > 24
}

onMounted(() => {
  window.addEventListener('scroll', handleNavScroll, { passive: true })
  handleNavScroll()

  sectionObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          activeSection.value = entry.target.id
        }
      })
    },
    { rootMargin: '-45% 0px -50% 0px', threshold: 0 }
  )
  navItems.forEach((item) => {
    const el = document.getElementById(item.id)
    if (el) sectionObserver.observe(el)
  })
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleNavScroll)
  if (sectionObserver) sectionObserver.disconnect()
})

/* ---------- Scroll reveal (fun pop-in on scroll) ---------- */
let revealObserver = null

onMounted(() => {
  const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches

  if (prefersReducedMotion) {
    document.querySelectorAll('.reveal, .reveal-scale').forEach((el) => el.classList.add('is-visible'))
    return
  }

  revealObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          entry.target.classList.add('is-visible')
          revealObserver.unobserve(entry.target)
        }
      })
    },
    { threshold: 0.15, rootMargin: '0px 0px -8% 0px' }
  )

  document.querySelectorAll('.reveal, .reveal-scale').forEach((el) => revealObserver.observe(el))
})

onUnmounted(() => {
  if (revealObserver) revealObserver.disconnect()
})
</script>

<template>
  <div class="folio">

    <nav class="navbar" :class="{ 'is-scrolled': navScrolled }">
      <div class="navbar-inner">
        <a href="#home" class="navbar-logo" @click.prevent="scrollToSection('home')">
          Niken<span class="navbar-logo-dot">.</span>
        </a>

        <ul class="navbar-links" :class="{ 'is-open': mobileMenuOpen }">
          <li v-for="item in navItems" :key="item.id">
            <a
              href="#"
              class="navbar-link"
              :class="{ 'is-active': activeSection === item.id }"
              @click.prevent="scrollToSection(item.id)"
            >
              {{ item.label }}
            </a>
          </li>
        </ul>

        <button
          class="navbar-toggle"
          :class="{ 'is-open': mobileMenuOpen }"
          @click="mobileMenuOpen = !mobileMenuOpen"
          aria-label="Toggle navigation menu"
        >
          <span></span>
          <span></span>
          <span></span>
        </button>
      </div>

      <div
        class="navbar-backdrop"
        :class="{ 'is-open': mobileMenuOpen }"
        @click="mobileMenuOpen = false"
      ></div>
    </nav>

    <div class="grid-overlay" aria-hidden="true"></div>
    <div class="glow" aria-hidden="true"></div>
    <img src="/img/blackwidow.png" class="brand-mark" alt="" aria-hidden="true" draggable="false" />


    <section class="hero" id="home">
      <div
        class="swing-char"
        :class="{ 'is-dragging': isDragging, 'is-entering': !hasEntered }"
        :style="swingStyle"
        ref="swingRef"
        @pointerdown="onPointerDown"
      >
        <div class="web-line"></div>
        <img src="/img/spiderman.png" class="char-img" draggable="false" alt="" />
      </div>

      <h1 class="hero-title">
        <span class="hero-title-label">Not a genius, billionare, playgirl, philantropist <br> but definitely a knowledge seeker</span>
        <span class="hero-title-outline">Hi! i'm Niken</span>
        <span class="hero-title-outline">Maharani</span>
        <span class="hero-title-line">Junior Web Developer & Tech enthusiast</span>
      </h1>

      <div class="hero-info">
      </div>

    </section>

    <section class="aboutme" id="about">
      <div class="glow glow--tertiary" aria-hidden="true"></div>
      <div class="aboutme-grid">
        <div class="aboutme-text">
          <span class="section-label reveal">About Me</span>
          <h2 class="aboutme-title reveal" style="transition-delay: 0.08s">
            Okey lets do this<br />one last time yeah
          </h2>
          <p class="aboutme-body reveal" style="transition-delay: 0.16s">
            My name is Niken Putri Maharani, a vocational high school student exploring the world of web development. Beyond crafting web applications, I’m fascinated by IoT and how software interacts with the physical world. Actively learning, building, and ready for new challenges! </p>

          <div class="aboutme-lower">
            <div class="aboutme-facts reveal" style="transition-delay: 0.24s">
              <div class="aboutme-fact">
                <span class="aboutme-fact-value">3</span>
                <span class="aboutme-fact-label">Years Learning</span>
              </div>
              <div class="aboutme-fact">
                <span class="aboutme-fact-value">5+</span>
                <span class="aboutme-fact-label">Tech Stacks</span>
              </div>
              <div class="aboutme-fact">
                <span class="aboutme-fact-value">∞</span>
                <span class="aboutme-fact-label">Curiosity</span>
              </div>
            </div>
          </div>

          <div class="aboutme-history">
            <div class="aboutme-edu">
              <span class="aboutme-edu-title">Education</span>
              <ul class="edu-timeline">
                <li
                  class="edu-timeline-item reveal"
                  v-for="(e, i) in educations"
                  :key="'edu-' + i"
                  :style="{ transitionDelay: (i * 0.1) + 's' }"
                >
                  <span class="edu-timeline-dot" aria-hidden="true"></span>
                  <span class="edu-timeline-year">{{ e.year }}</span>
                  <h4 class="edu-timeline-role">{{ e.role }}</h4>
                  <span class="edu-timeline-place">{{ e.place }}</span>
                </li>
              </ul>
            </div>

            <div class="aboutme-edu aboutme-exp">
              <span class="aboutme-edu-title">Experience</span>
              <ul class="edu-timeline">
                <li
                  class="edu-timeline-item reveal"
                  v-for="(x, i) in experiences"
                  :key="'exp-' + i"
                  :style="{ transitionDelay: (i * 0.1) + 's' }"
                >
                  <span class="edu-timeline-dot" aria-hidden="true"></span>
                  <span class="edu-timeline-year">{{ x.year }}</span>
                  <h4 class="edu-timeline-role">{{ x.role }}</h4>
                  <span class="edu-timeline-place">{{ x.place }}</span>
                </li>
              </ul>
            </div>
          </div>
        </div>

        <div class="aboutme-photo reveal-scale">
          <div class="aboutme-photo-ring" aria-hidden="true"></div>
          <div class="aboutme-photo-frame">
            <img src="/img/gw.jpeg" alt="Niken Putri Maharani" class="aboutme-photo-img" />
            <div class="aboutme-photo-fallback" aria-hidden="true">NM</div>
          </div>
          <span class="aboutme-photo-caption">Niken Putri Maharani</span>
        </div>
      </div>
    </section>


    <section class="skills" id="skills">
      <div class="glow glow--secondary" aria-hidden="true"></div>
      <div class="skills-top">
      </div>

      <h2 class="skills-title reveal">My Skills</h2>

      <div class="orbit-wrap reveal-scale" style="transition-delay: 0.1s">
        <div class="orbit-track"></div>

        <div class="orbit-core">
          <div class="orbit-core-glow" aria-hidden="true"></div>
            <img
              class="orbit-core-img"
              src="/img/skills/reactor.png"
              alt=""
              @error="handleImgError"
            />
            <div class="orbit-core-fallback" style="display:none;">
              <span>NM</span>
              <span>My Stack</span>
            </div>

        </div>

        <div
          v-for="item in orbitItems"
          :key="item.id"
          class="orbit-pivot"
          :style="{
            '--duration': ORBIT_DURATION + 's',
            '--delay': item.delay + 's',
            transform: `rotate(${item.angle}deg)`
          }"
        >
          <div class="orbit-arm" :style="{ '--enter-delay': item.enterDelay + 's' }">
            <div
              class="orbit-counter"
              :style="{
                '--duration': ORBIT_DURATION + 's',
                '--delay': item.delay + 's',
                transform: `rotate(${-item.angle}deg)`
              }"
            >
              <div class="orbit-badge" :class="{ 'orbit-badge--lab': item.lab }" tabindex="0">
                <img :src="item.img" :alt="item.name" @error="handleImgError" />
                <span class="orbit-badge-fallback" style="display:none;">{{ item.initials }}</span>
              </div>
              <span class="orbit-name">{{ item.name }}</span>
              <span class="orbit-meta">
                <span v-if="item.lab" class="status-dot" aria-hidden="true"></span>
                {{ item.meta }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <div class="skills-legend">
      </div>
    </section>

    <section class="projects" id="potatoes">
      <div class="glow glow--tertiary" aria-hidden="true"></div>

      <div class="section-top reveal">
        <span class="eyebrow">There's my big baby potatos stuff</span>
        <span class="eyebrow eyebrow--right">2024 — 2026</span>
      </div>

      <h2 class="projects-title reveal" style="transition-delay: 0.08s">My Big Potatoes</h2>

      <div class="stuff-tabbar reveal" style="transition-delay: 0.16s">
        <button
          v-for="tab in stuffTabs"
          :key="tab.id"
          class="stuff-tab"
          :class="{ 'is-active': activeStuffTab === tab.id }"
          @click="activeStuffTab = tab.id"
        >
          {{ tab.label }}
        </button>
        <span
          class="stuff-tab-indicator"
          :style="{ transform: `translateX(${stuffTabs.findIndex(t => t.id === activeStuffTab) * 100}%)` }"
        ></span>
      </div>

      <transition name="stuff-fade" mode="out-in">
        <div v-if="activeStuffTab === 'projects'" key="projects" class="stuff-grid">
          <article
            v-for="(p, i) in stuffProjects"
            :key="p.title + i"
            class="stuff-card stuff-pop"
            :style="{ animationDelay: (i % 6) * 0.06 + 's' }"
          >
            <button
              class="stuff-card-image"
              type="button"
              :aria-label="`Lihat preview ${p.title} lebih besar`"
              @click.stop="openLightbox(p.img, p.title)"
            >
              <img :src="p.img" :alt="p.title" @error="handleImgError" />
              <div class="stuff-card-image-fallback" style="display:none;">
                {{ p.title }}
              </div>
              <span class="stuff-card-zoom" aria-hidden="true">⤢</span>
            </button>

            <div class="stuff-card-body">
              <h3 class="stuff-card-title">{{ p.title }}</h3>
              <p v-if="p.desc" class="stuff-card-desc">{{ p.desc }}</p>
              <span class="stuff-card-tags">{{ p.tags }}</span>
              <button class="stuff-card-btn" type="button">
                Details
                <span class="stuff-card-btn-arrow" aria-hidden="true">→</span>
              </button>
            </div>
          </article>
        </div>

        <div v-else-if="activeStuffTab === 'certificates'" key="certificates" class="stuff-grid">
          <article
            v-for="(c, i) in stuffCertificates"
            :key="c.issuer + i"
            class="stuff-card stuff-card--cert stuff-pop"
            :style="{ animationDelay: (i % 6) * 0.06 + 's' }"
          >
            <button
              class="stuff-card-image stuff-card-image--cert"
              type="button"
              :aria-label="`Lihat sertifikat ${c.title} lebih besar`"
              @click="openLightbox(c.img, c.title)"
            >
              <img :src="c.img" :alt="c.title" @error="handleImgError" />
              <div class="stuff-card-image-fallback" style="display:none;">
                {{ c.title }}
              </div>
              <span class="stuff-card-zoom" aria-hidden="true">⤢</span>
            </button>

            <div class="stuff-card-body">
              <h3 class="stuff-card-title">{{ c.title }}</h3>
              <span class="stuff-card-tags">{{ c.issuer }} · {{ c.year }}</span>
            </div>
          </article>
        </div>

        <div v-else key="techstack" class="stuff-grid stuff-grid--tech">
          <article
            v-for="(t, i) in stuffTechStack"
            :key="t.name"
            class="stuff-chip stuff-pop"
            :style="{ animationDelay: (i % 6) * 0.05 + 's' }"
          >
            <span class="stuff-chip-name">{{ t.name }}</span>
            <span class="stuff-chip-category">{{ t.category }}</span>
          </article>
        </div>
      </transition>
    </section>

    <section class="contact" id="contact">
      <div class="glow glow--secondary" aria-hidden="true"></div>
      <span class="eyebrow reveal">Let's Work</span>
      <h2 class="contact-title reveal" style="transition-delay: 0.08s">
        Got a project in mind?<br />
        <span class="contact-title-outline">Let's talk.</span>
      </h2>
      <a href="mailto:hello@example.com" class="contact-email reveal" style="transition-delay: 0.16s">ikeenn158@gmail.com</a>

      <div class="contact-links reveal" style="transition-delay: 0.24s">
        <a href="#" class="contact-link">LinkedIn</a>
        <a href="#" class="contact-link">GitHub</a>
        <a href="#" class="contact-link">Instagram</a>
      </div>
    </section>

    <footer class="site-footer">
      <span class="footer-name">Niken Putri Maharani</span>
      <span class="footer-copy">© 2026 — U </span>
      <a href="#" class="footer-top">Back to top ↑</a>
    </footer>

    <transition name="lightbox-fade">
      <div
        v-if="lightboxImage"
        class="lightbox-overlay"
        @click="closeLightbox"
      >
        <button class="lightbox-close" type="button" aria-label="Tutup" @click="closeLightbox">✕</button>
        <img
          :src="lightboxImage"
          :alt="lightboxAlt"
          class="lightbox-img"
          @click.stop
        />
        <span v-if="lightboxAlt" class="lightbox-caption">{{ lightboxAlt }}</span>
      </div>
    </transition>

  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.folio {
  position: relative;
  background: linear-gradient(135deg, #2a0506 0%, #140503 32%, #0a0706 60%, #0a0706 100%);
  color: #f5efe6;
  font-family: 'Manrope', sans-serif;
  overflow: hidden;
  max-width: 100vw;
}

/* ============================================================
   SCROLL REVEAL — fun springy pop-in as sections enter view
   ============================================================ */
.reveal {
  opacity: 0;
  transform: translateY(28px);
  transition: opacity 0.7s cubic-bezier(0.16, 1, 0.3, 1),
    transform 0.7s cubic-bezier(0.34, 1.56, 0.64, 1);
  will-change: opacity, transform;
}

.reveal.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.reveal-scale {
  opacity: 0;
  transform: scale(0.82) translateY(16px);
  transition: opacity 0.7s cubic-bezier(0.16, 1, 0.3, 1),
    transform 0.85s cubic-bezier(0.34, 1.56, 0.64, 1);
  will-change: opacity, transform;
}

.reveal-scale.is-visible {
  opacity: 1;
  transform: scale(1) translateY(0);
}

@media (prefers-reduced-motion: reduce) {
  .reveal,
  .reveal-scale {
    opacity: 1 !important;
    transform: none !important;
    transition: none !important;
  }
}

/* Cards inside the "Big Potatoes" tabs are destroyed/recreated on every tab
   switch, so they get their own self-playing entrance animation instead of
   the scroll IntersectionObserver — avoids fighting with the tab's own fade
   transition. */
@keyframes stuffPop {
  from {
    opacity: 0;
    transform: translateY(18px) scale(0.96);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.stuff-pop {
  animation: stuffPop 0.55s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}

@media (prefers-reduced-motion: reduce) {
  .stuff-pop {
    animation: none;
  }
}

/* ============================================================
   NAVBAR
   ============================================================ */
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 50;
  padding: 18px 64px;
  background: rgba(10, 7, 6, 0.35);
  backdrop-filter: blur(0px);
  -webkit-backdrop-filter: blur(0px);
  border-bottom: 1px solid rgba(245, 239, 230, 0);
  transition: background 0.35s ease, border-color 0.35s ease, backdrop-filter 0.35s ease, padding 0.35s ease;
}

.navbar.is-scrolled {
  padding: 14px 64px;
  background: rgba(10, 7, 6, 0.72);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-bottom: 1px solid rgba(232, 201, 160, 0.14);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.35);
}

.navbar-inner {
  position: relative;
  z-index: 2;
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.navbar-logo {
  font-family: 'Bodoni Moda', serif;
  font-weight: 600;
  font-style: italic;
  font-size: 20px;
  color: #f5efe6;
  text-decoration: none;
  letter-spacing: 0.02em;
}

.navbar-logo-dot {
  color: #b31217;
}

.navbar-links {
  display: flex;
  align-items: center;
  gap: 6px;
  list-style: none;
  margin: 0;
  padding: 0;
}

.navbar-link {
  position: relative;
  display: inline-block;
  padding: 8px 16px;
  font-family: 'Manrope', sans-serif;
  font-weight: 600;
  font-size: 12px;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #cfc6b8;
  text-decoration: none;
  border-radius: 999px;
  transition: color 0.3s ease, background 0.3s ease, box-shadow 0.3s ease;
}

.navbar-link:hover {
  color: #f5efe6;
}

.navbar-link.is-active {
  color: #0a0706;
  background: linear-gradient(135deg, #e8c9a0, #cfa876);
  box-shadow: 0 4px 16px rgba(179, 18, 23, 0.35);
}

.navbar-toggle {
  display: none;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 5px;
  width: 36px;
  height: 36px;
  border: 1px solid rgba(232, 201, 160, 0.3);
  border-radius: 8px;
  background: rgba(245, 239, 230, 0.05);
  cursor: pointer;
  padding: 0;
  z-index: 2;
  position: relative;
}

.navbar-toggle span {
  display: block;
  width: 16px;
  height: 1.5px;
  background: #e8c9a0;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.navbar-toggle.is-open span:nth-child(1) {
  transform: translateY(6.5px) rotate(45deg);
}

.navbar-toggle.is-open span:nth-child(2) {
  opacity: 0;
}

.navbar-toggle.is-open span:nth-child(3) {
  transform: translateY(-6.5px) rotate(-45deg);
}

.navbar-backdrop {
  display: none;
}

@media (max-width: 900px) {
  .navbar {
    padding: 16px 28px;
  }

  .navbar.is-scrolled {
    padding: 12px 28px;
  }

  .navbar-toggle {
    display: flex;
  }

  .navbar-links {
    position: fixed;
    top: 0;
    right: 0;
    height: 100vh;
    width: min(78vw, 320px);
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
    gap: 6px;
    padding: 100px 36px 40px;
    background: rgba(10, 6, 6, 0.97);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    border-left: 1px solid rgba(232, 201, 160, 0.16);
    transform: translateX(100%);
    transition: transform 0.4s cubic-bezier(0.65, 0, 0.35, 1);
    z-index: 51;
  }

  .navbar-links.is-open {
    transform: translateX(0);
  }

  .navbar-link {
    width: 100%;
    padding: 12px 16px;
    font-size: 13px;
  }

  .navbar-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0);
    pointer-events: none;
    transition: background 0.35s ease;
    z-index: 40;
  }

  .navbar-backdrop.is-open {
    background: rgba(0, 0, 0, 0.45);
    pointer-events: auto;
  }
}

.grid-overlay {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(to right, rgba(245, 239, 230, 0.045) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(245, 239, 230, 0.045) 1px, transparent 1px);
  background-size: 120px 120px;
  pointer-events: none;
  z-index: 0;
}

.glow {
  position: absolute;
  top: -25%;
  left: -20%;
  width: 90vw;
  height: 90vw;
  max-width: 1100px;
  max-height: 1100px;
  background: radial-gradient(circle, rgba(179, 18, 23, 0.85) 0%, rgba(120, 12, 16, 0.4) 38%, rgba(10, 7, 6, 0) 72%);
  pointer-events: none;
  z-index: 0;
}

.glow--secondary {
  position: absolute;
  bottom: -30%;
  right: -15%;
  width: 70vw;
  height: 70vw;
  max-width: 900px;
  max-height: 900px;
  background: radial-gradient(circle, rgba(179, 18, 23, 0.55) 0%, rgba(179, 18, 23, 0) 70%);
  pointer-events: none;
  z-index: 0;
}

.brand-mark {
  position: absolute;
  top: 2%;
  left: 5%;
  width: 420px;
  max-width: 46vw;
  opacity: 0.35;  
  pointer-events: none;
  user-select: none;
  z-index: 0;
}

.side-marquee {
  position: fixed;
  right: 18px;
  top: 0;
  bottom: 0;
  writing-mode: vertical-rl;
  display: flex;
  align-items: center;
  font-family: 'Manrope', sans-serif;
  font-size: 11px;
  letter-spacing: 0.25em;
  color: rgba(232, 201, 160, 0.55);
  z-index: 5;
  pointer-events: none;
}

.eyebrow {
  font-size: 12px;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  font-style: italic;
  color: #cfc6b8;
}

.section-label {
  display: inline-block;
  font-size: 13px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: #e8c9a0;
  border-bottom: 1px solid rgba(232, 201, 160, 0.35);
  padding-bottom: 10px;
  margin-bottom: 22px;
}

.hero {
  position: relative;
  z-index: 1;
  padding: 120px 64px 90px;
  max-width: 1400px;
  margin: 0 auto;
  overflow: visible;
}

.swing-char {
  position: absolute;
  top: -20px;
  left: 24px;
  width: 230px;
  transform-origin: top center;
  animation: swing 3.2s ease-in-out infinite;
  z-index: 2;
  cursor: grab;
  touch-action: none;
  user-select: none;
}

.swing-char.is-dragging {
  cursor: grabbing;
}

.web-line {
  width: 1px;
  height: 220px;
  margin: 0 auto;
  background: linear-gradient(to bottom, rgba(245, 239, 230, 0), rgba(245, 239, 230, 0.55));
  pointer-events: none;
  transform-origin: top;
}

.char-img {
  width: 100%;
  display: block;
  filter: drop-shadow(0 12px 20px rgba(0, 0, 0, 0.45));
  pointer-events: none;
}

@keyframes swing {
  0%, 100% { transform: rotate(-10deg); }
  50% { transform: rotate(10deg); }
}

.swing-char.is-entering {
  animation: dropIn 1.05s cubic-bezier(0.34, 1.42, 0.4, 1) both;
}

.swing-char.is-entering .web-line {
  animation: webStretch 1.05s cubic-bezier(0.34, 1.42, 0.4, 1) both;
}

.swing-char.is-entering .char-img {
  animation: charFadeIn 0.5s ease-out both;
  animation-delay: 0.15s;
}

@keyframes dropIn {
  0%   { transform: translateY(-240px) rotate(-10deg); }
  100% { transform: translateY(0) rotate(-10deg); }
}

@keyframes webStretch {
  0%   { transform: scaleY(0); opacity: 0; }
  35%  { opacity: 1; }
  100% { transform: scaleY(1); opacity: 1; }
}

@keyframes charFadeIn {
  0%   { opacity: 0; }
  100% { opacity: 1; }
}

@media (max-width: 900px) {
  .swing-char {
    width: 150px;
    left: 12px;
  }

  .web-line {
    height: 150px;
  }

  .brand-mark {
    width: 320px;
    max-width: 65vw;
    top: -1%;
    left: -12%;
    opacity: 0.13;
  }
}

@media (max-width: 560px) {
  .swing-char {
    width: 110px;
    top: -10px;
  }

  .web-line {
    height: 110px;
  }

  .brand-mark {
    width: 220px;
    max-width: 70vw;
    opacity: 0.11;
  }
}

.hero-top {
  display: flex;
  justify-content: space-between;
  margin-bottom: 60px;
}

.eyebrow--right {
  margin-left: auto;
}

.hero-title {
  margin: 0 0 40px;
  line-height: 0.9;
  text-align: right;
}

.hero-title-label {
  display: block;
  font-family: 'Bodoni Moda', serif;
  font-style: italic;
  font-weight: 500;
  font-size: 19px;
  letter-spacing: 0.03em;
  color: #e0b49a;
  margin-bottom: 10px;
}

.hero-title-outline {
  display: block;
  font-family: 'Bodoni Moda', serif;
  font-weight: 600;
  font-size: clamp(38px, 10vw, 100px);
  color: transparent;
  -webkit-text-stroke: 1.5px #f5efe6;
  letter-spacing: -0.01em;
}

.hero-title-line {
  display: block;
  font-family: 'Manrope', sans-serif;
  font-weight: 600;
  font-size: 14px;
  letter-spacing: 0.35em;
  text-transform: uppercase;
  color: #e8c9a0;
  margin-top: 26px;
  padding-top: 18px;
  position: relative;
}

.hero-title-line::before {
  content: '';
  position: absolute;
  top: 0;
  right: 0;
  width: 72px;
  height: 1px;
  background: linear-gradient(90deg, rgba(179, 18, 23, 0), #b31217);
}

.hero-info {
  display: flex;
  gap: 32px;
  margin-bottom: 90px;
  flex-wrap: wrap;
}

.hero-contact,
.hero-behance {
  margin: 0;
  font-size: 14px;
  color: #cfc6b8;
  display: flex;
  align-items: center;
  gap: 8px;
}

.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #b31217;
  box-shadow: 0 0 10px #b31217;
}

.aboutme {
  position: relative;
  z-index: 1;
  padding: 48px 64px 110px;
  max-width: 1400px;
  margin: 0 auto;
  border-top: 1px solid rgba(245, 239, 230, 0.08);
}

.aboutme-grid {
  display: grid;
  grid-template-columns: 1.3fr 0.7fr;
  gap: 64px;
  align-items: center;
}

.aboutme-title {
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: clamp(30px, 4vw, 48px);
  line-height: 1.15;
  margin: 0 0 26px;
}

.aboutme-body {
  margin: 0 0 36px;
  font-size: 16px;
  line-height: 1.8;
  color: #cfc6b8;
  max-width: 58ch;
}

/* ---------- about lower row: facts ---------- */
.aboutme-lower {
  display: flex;
  gap: 48px;
  flex-wrap: wrap;
  align-items: flex-start;
}

.aboutme-facts {
  display: flex;
  gap: 40px;
  flex-wrap: wrap;
  flex: 0 0 auto;
}

.aboutme-fact {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.aboutme-fact-value {
  font-family: 'Bodoni Moda', serif;
  font-weight: 600;
  font-size: 32px;
  color: #e8c9a0;
  line-height: 1;
}

.aboutme-fact-label {
  font-size: 11px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #cfc6b8;
}

/* ---------- education + experience side-by-side ---------- */
.aboutme-history {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 48px;
  margin-top: 40px;
}

.aboutme-edu {
  flex: 1 1 220px;
  min-width: 220px;
  padding-left: 32px;
  border-left: 1px solid rgba(245, 239, 230, 0.08);
}

.aboutme-edu-title {
  display: block;
  font-size: 11px;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #e8c9a0;
  margin-bottom: 16px;
}

.edu-timeline {
  list-style: none;
  margin: 0;
  padding: 0;
  position: relative;
}

.edu-timeline-item {
  position: relative;
  padding-left: 22px;
  padding-bottom: 16px;
}

.edu-timeline-item:not(:last-child)::before {
  content: '';
  position: absolute;
  left: 4px;
  top: 14px;
  bottom: -2px;
  width: 1px;
  background: rgba(245, 239, 230, 0.12);
}

.edu-timeline-item:last-child {
  padding-bottom: 0;
}

.edu-timeline-dot {
  position: absolute;
  left: 0;
  top: 5px;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: #b31217;
  box-shadow: 0 0 10px #b31217;
}

.edu-timeline-year {
  display: block;
  font-size: 11px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #cfc6b8;
  margin-bottom: 3px;
}

.edu-timeline-role {
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: 16px;
  margin: 0 0 2px;
  line-height: 1.2;
}

.edu-timeline-place {
  font-size: 12px;
  color: #cfc6b8;
}

.aboutme-photo {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-self: center;
}

.aboutme-photo-ring {
  position: absolute;
  inset: -18px;
  border-radius: 50%;
  border: 1px dashed rgba(232, 201, 160, 0.35);
  animation: ringSpin 26s linear infinite;
}

@keyframes ringSpin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.aboutme-photo-frame {
  position: relative;
  width: 260px;
  aspect-ratio: 1;
  border-radius: 50%;
  padding: 8px;
  background: linear-gradient(160deg, rgba(40, 10, 10, 0.55), rgba(10, 6, 6, 0.4));
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(232, 201, 160, 0.4);
  box-shadow:
    0 0 0 1px rgba(179, 18, 23, 0.15) inset,
    0 0 45px rgba(179, 18, 23, 0.3),
    0 20px 50px rgba(0, 0, 0, 0.4);
  overflow: hidden;
}

.aboutme-photo-img {
  position: relative;
  z-index: 1;
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
  filter: grayscale(0.1) contrast(1.05);
}

.aboutme-photo-fallback {
  position: absolute;
  inset: 8px;
  z-index: 0;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: 64px;
  color: transparent;
  -webkit-text-stroke: 1.5px #e8c9a0;
  background: radial-gradient(circle, rgba(179, 18, 23, 0.25), rgba(10, 6, 6, 0.6));
}

.aboutme-photo-caption {
  margin-top: 22px;
  font-size: 12px;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #cfc6b8;
}

.about {
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 56px;
  align-items: start;
}

.about-photo-frame {
  width: 100%;
  aspect-ratio: 1;
  border-radius: 50%;
  border: 1px solid rgba(232, 201, 160, 0.5);
  padding: 10px;
  overflow: hidden;
}

.about-photo-frame img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
  filter: grayscale(0.15) contrast(1.05);
}

.about-text p {
  margin: 0;
  font-size: 16px;
  line-height: 1.75;
  color: #cfc6b8;
  max-width: 62ch;
}

.about-text strong {
  color: #f5efe6;
}

.skills {
  position: relative;
  z-index: 1;
  padding: 48px 64px 90px;
  max-width: 1400px;
  margin: 0 auto;
  border-top: 1px solid rgba(245, 239, 230, 0.08);
}

.skills-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 40px;
}

.skills-meta {
  display: flex;
  flex-direction: column;
  font-size: 13px;
  color: #cfc6b8;
}

.skills-meta--right {
  text-align: right;
}

.skills-meta-name {
  font-weight: 600;
  color: #f5efe6;
}

.skills-title {
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: clamp(28px, 4vw, 44px);
  line-height: 1.05;
  margin: 0 0 32px;
}

/* ============ SKILLS ORBIT ============ */
.orbit-wrap {
  position: relative;
  width: min(600px, 92vw);
  aspect-ratio: 1 / 1;
  margin: 0 auto;
}

.orbit-track {
  position: absolute;
  inset: 8%;
  border-radius: 50%;
  border: 1px dashed rgba(232, 201, 160, 0.16);
  pointer-events: none;
}

/* ---- Core / arc reactor ---- */
.orbit-core {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 32%;
  aspect-ratio: 1 / 1;
  border-radius: 50%;
  z-index: 3;
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}

.orbit-core-glow {
  position: absolute;
  inset: -30%;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(18, 61, 179, 0.55) 0%, rgba(18, 90, 179, 0.12) 45%, rgba(18, 56, 179, 0) 72%);
  animation: corePulse 3.2s ease-in-out infinite;
}

@keyframes corePulse {
  0%, 100% { opacity: 0.65; transform: scale(0.94); }
  50% { opacity: 1; transform: scale(1.05); }
}

.orbit-core-ring {
  position: absolute;
  inset: 6%;
  border-radius: 50%;
  background: conic-gradient(from 0deg, rgba(232, 201, 160, 0.9), rgba(179, 18, 23, 0.9), rgba(232, 201, 160, 0.9));
  -webkit-mask: radial-gradient(farthest-side, transparent calc(100% - 3px), #000 calc(100% - 3px));
          mask: radial-gradient(farthest-side, transparent calc(100% - 3px), #000 calc(100% - 3px));
  animation: coreSpin 6s linear infinite;
}

@keyframes coreSpin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.orbit-core-frame {
  position: relative;
  width: 82%;
  height: 82%;
  border-radius: 50%;
  padding: 6px;
  background: linear-gradient(160deg, rgba(40, 10, 10, 0.6), rgba(10, 6, 6, 0.5));
  border: 1px solid rgba(232, 201, 160, 0.45);
  box-shadow:
    0 0 0 1px rgba(179, 18, 23, 0.2) inset,
    0 0 40px rgba(179, 18, 23, 0.4),
    0 20px 45px rgba(0, 0, 0, 0.45);
  overflow: hidden;
}

.orbit-core-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
  display: block;
}

.orbit-core-fallback {
  position: absolute;
  inset: 6px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 2px;
  background: radial-gradient(circle, rgba(179, 18, 23, 0.3), rgba(10, 6, 6, 0.7));
  font-family: 'Bodoni Moda', serif;
}

.orbit-core-fallback span:first-child {
  font-size: clamp(22px, 4vw, 30px);
  font-weight: 600;
  color: transparent;
  -webkit-text-stroke: 1.4px #e8c9a0;
}

.orbit-core-fallback span:last-child {
  font-family: 'Manrope', sans-serif;
  font-size: 9px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: #cfc6b8;
}

/* ---- Orbiting skills: pivot rotates the whole radius, counter cancels it out ---- */
.orbit-pivot {
  position: absolute;
  inset: 0;
  pointer-events: none;
  animation: orbitSpin var(--duration, 48s) linear infinite;
  animation-delay: var(--delay, 0s);
}

.orbit-arm {
  position: absolute;
  top: calc(40% - 31%);
  left: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
  --orbit-radius: 160px;
  opacity: 0;
}

.orbit-wrap.is-visible .orbit-arm {
  animation: orbitArmEnter 0.85s cubic-bezier(0.34, 1.56, 0.64, 1) both;
  animation-delay: var(--enter-delay, 0s);
}

@keyframes orbitArmEnter {
  0% {
    opacity: 0;
    transform: translate(-50%, -50%) translateY(var(--orbit-radius)) scale(0.15);
  }
  55% {
    opacity: 1;
  }
  100% {
    opacity: 1;
    transform: translate(-50%, -50%) translateY(0) scale(1);
  }
}

@media (max-width: 640px) {
  .orbit-arm {
    --orbit-radius: 95px;
  }
}

.orbit-counter {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  width: 116px;
  pointer-events: auto;
  animation: orbitCounterSpin var(--duration, 48s) linear infinite;
  animation-delay: var(--delay, 0s);
}

@keyframes orbitSpin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes orbitCounterSpin {
  from { transform: rotate(0deg); }
  to { transform: rotate(-360deg); }
}

.orbit-badge {
  position: relative;
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: none;
  box-shadow: none;
  transition: transform 0.35s ease;
  cursor: pointer;
  outline: none;
}

.orbit-badge img {
  width: 78%;
  height: 78%;
  object-fit: contain;
  filter:
    drop-shadow(0 0 6px rgba(179, 18, 23, 0.9))
    drop-shadow(0 0 16px rgba(179, 18, 23, 0.6))
    drop-shadow(0 0 30px rgba(179, 18, 23, 0.32));
  transition: filter 0.35s ease, transform 0.35s ease;
}

.orbit-badge--lab img {
  filter:
    drop-shadow(0 0 6px rgba(232, 201, 160, 0.85))
    drop-shadow(0 0 16px rgba(232, 201, 160, 0.5))
    drop-shadow(0 0 28px rgba(232, 201, 160, 0.28));
}

.orbit-badge-fallback {
  font-family: 'Bodoni Moda', serif;
  font-weight: 600;
  font-size: 20px;
  color: #e8c9a0;
  text-shadow:
    0 0 8px rgba(179, 18, 23, 0.9),
    0 0 20px rgba(179, 18, 23, 0.55);
}

.orbit-badge--lab .orbit-badge-fallback {
  text-shadow:
    0 0 8px rgba(232, 201, 160, 0.85),
    0 0 20px rgba(232, 201, 160, 0.5);
}

.orbit-name {
  margin-top: 10px;
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: 13.5px;
  color: #f5efe6;
  line-height: 1.15;
}

.orbit-meta {
  margin-top: 3px;
  font-size: 10px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #e8c9a0;
  display: inline-flex;
  align-items: center;
  gap: 5px;
}

.orbit-meta .status-dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: #e8c9a0;
  box-shadow: 0 0 8px #e8c9a0;
  animation: statusPulse 1.6s ease-in-out infinite;
}

@keyframes statusPulse {
  0%, 100% { opacity: 0.35; transform: scale(0.85); }
  50% { opacity: 1; transform: scale(1); }
}

.orbit-pivot:has(.orbit-badge:hover),
.orbit-pivot:has(.orbit-badge:focus-visible) {
  animation-play-state: paused;
}

.orbit-pivot:has(.orbit-badge:hover) .orbit-counter,
.orbit-pivot:has(.orbit-badge:focus-visible) .orbit-counter {
  animation-play-state: paused;
}

.orbit-badge:hover,
.orbit-badge:focus-visible {
  transform: scale(1.16);
}

.orbit-badge:hover img,
.orbit-badge:focus-visible img {
  filter:
    drop-shadow(0 0 10px rgba(179, 18, 23, 1))
    drop-shadow(0 0 26px rgba(179, 18, 23, 0.8))
    drop-shadow(0 0 48px rgba(232, 201, 160, 0.5));
}

.orbit-badge--lab:hover img,
.orbit-badge--lab:focus-visible img {
  filter:
    drop-shadow(0 0 10px rgba(232, 201, 160, 1))
    drop-shadow(0 0 24px rgba(232, 201, 160, 0.7))
    drop-shadow(0 0 42px rgba(232, 201, 160, 0.4));
}

@media (prefers-reduced-motion: reduce) {
  .orbit-pivot,
  .orbit-counter,
  .orbit-core-ring,
  .orbit-core-glow,
  .orbit-arm {
    animation: none !important;
  }

  .orbit-arm {
    opacity: 1;
  }
}

.skills-legend {
  position: relative;
  z-index: 2;
  display: flex;
  justify-content: center;
  gap: 28px;
  margin-top: 54px;
  font-size: 11px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #cfc6b8;
}

.skills-legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.skills-legend-dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
}

.skills-legend-dot--core {
  background: linear-gradient(135deg, #e8c9a0, #b31217);
}

.skills-legend-dot--lab {
  border: 1px dashed #e8c9a0;
}

@media (max-width: 640px) {
  .orbit-counter { width: 84px; }
  .orbit-badge { width: 48px; height: 48px; }
  .orbit-name { font-size: 11px; }
  .orbit-meta { font-size: 9px; }
}


/* ============================================================
   SECTION LANJUTAN (My Big Potatoes, Contact, Footer)
   ============================================================ */

.section-top {
  display: flex;
  justify-content: space-between;
  margin-bottom: 60px;
}

.glow--tertiary {
  position: absolute;
  top: 10%;
  left: 40%;
  width: 60vw;
  height: 60vw;
  max-width: 800px;
  max-height: 800px;
  background: radial-gradient(circle, rgba(179, 18, 23, 0.35) 0%, rgba(179, 18, 23, 0) 70%);
  pointer-events: none;
  z-index: 0;
}

/* ---------- projects / my big potatoes ---------- */
.projects {
  position: relative;
  z-index: 1;
  padding: 48px 64px 110px;
  max-width: 1400px;
  margin: 0 auto;
  border-top: 1px solid rgba(245, 239, 230, 0.08);
}

.projects-title {
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: clamp(38px, 6.5vw, 64px);
  margin: 0 auto 50px;
  text-align: center;
}

/* ---------- stuff tabbar ---------- */
.stuff-tabbar {
  position: relative;
  display: flex;
  padding: 7px;
  border-radius: 999px;
  background: rgba(245, 239, 230, 0.05);
  border: 1px solid rgba(232, 201, 160, 0.16);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  margin: 0 auto 50px;
  max-width: 640px;
}

.stuff-tab {
  position: relative;
  z-index: 1;
  flex: 1;
  padding: 15px 24px;
  border: none;
  background: transparent;
  font-family: 'Manrope', sans-serif;
  font-weight: 600;
  font-size: 13px;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  color: #cfc6b8;
  cursor: pointer;
  transition: color 0.35s ease;
  white-space: nowrap;
}

.stuff-tab.is-active {
  color: #0a0706;
}

.stuff-tab-indicator {
  position: absolute;
  top: 7px;
  left: 7px;
  bottom: 7px;
  width: calc(33.333% - 5px);
  border-radius: 999px;
  background: linear-gradient(135deg, #e8c9a0, #cfa876);
  box-shadow: 0 4px 18px rgba(179, 18, 23, 0.35);
  transition: transform 0.4s cubic-bezier(0.65, 0, 0.35, 1);
  z-index: 0;
}

/* ---------- stuff grid ---------- */
.stuff-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

/* ---------- project cards (with image) ---------- */
.stuff-card {
  position: relative;
  display: flex;
  flex-direction: column;
  border-radius: 20px;
  background: linear-gradient(160deg, rgba(40, 10, 10, 0.45), rgba(10, 6, 6, 0.35));
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border: 1px solid rgba(232, 201, 160, 0.14);
  box-shadow:
    0 0 0 1px rgba(179, 18, 23, 0.12) inset,
    0 20px 50px rgba(0, 0, 0, 0.35);
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.4s cubic-bezier(0.16, 1, 0.3, 1), border-color 0.4s ease, box-shadow 0.4s ease;
}

.stuff-card:hover {
  transform: translateY(-6px);
  border-color: rgba(179, 18, 23, 0.55);
  box-shadow:
    0 0 0 1px rgba(179, 18, 23, 0.3) inset,
    0 0 40px rgba(179, 18, 23, 0.25),
    0 24px 60px rgba(0, 0, 0, 0.45);
}

/* image area on top of the card */
.stuff-card-image {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 10;
  overflow: hidden;
  background: rgba(245, 239, 230, 0.04);
  border-bottom: 1px solid rgba(232, 201, 160, 0.14);
}

.stuff-card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.5s cubic-bezier(0.16, 1, 0.3, 1);
}

.stuff-card:hover .stuff-card-image img {
  transform: scale(1.05);
}

.stuff-card-image-fallback {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 12px;
  text-align: center;
  font-family: 'Bodoni Moda', serif;
  font-style: italic;
  font-size: 14px;
  color: #cfc6b8;
  background: radial-gradient(circle, rgba(179, 18, 23, 0.18), rgba(10, 6, 6, 0.6));
}

/* body / text area below the image */
.stuff-card-body {
  padding: 24px 24px 26px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.stuff-card-num {
  display: block;
  font-family: 'Bodoni Moda', serif;
  font-style: italic;
  font-size: 13px;
  color: #b31217;
  margin-bottom: 4px;
}

.stuff-card-icon {
  display: block;
  font-size: 20px;
  color: #e8c9a0;
  margin-bottom: 18px;
}

.stuff-card-title {
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: 19px;
  margin: 0;
  line-height: 1.25;
}

.stuff-card-desc {
  margin: 0;
  font-size: 13.5px;
  line-height: 1.6;
  color: #cfc6b8;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.stuff-card-tags {
  display: block;
  font-size: 11px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #e8c9a0;
}

.stuff-card-btn {
  align-self: flex-start;
  margin-top: 8px;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  border-radius: 999px;
  border: 1px solid rgba(232, 201, 160, 0.3);
  background: rgba(245, 239, 230, 0.06);
  color: #f5efe6;
  font-family: 'Manrope', sans-serif;
  font-weight: 600;
  font-size: 12px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  cursor: pointer;
  transition: background 0.3s ease, border-color 0.3s ease, transform 0.3s ease;
}

.stuff-card-btn-arrow {
  transition: transform 0.3s ease;
}

.stuff-card:hover .stuff-card-btn {
  background: linear-gradient(135deg, #e8c9a0, #cfa876);
  border-color: transparent;
  color: #0a0706;
}

.stuff-card:hover .stuff-card-btn-arrow {
  transform: translateX(4px);
}

/* certificate cards: image on top (clickable to enlarge) + body below */
.stuff-card--cert {
  padding: 0;
}

.stuff-card-image--cert {
  aspect-ratio: 4 / 3;
  background: #eceff3;
}

/* image button reset (used by both project and certificate cards) */
.stuff-card-image {
  display: block;
  width: 100%;
  padding: 0;
  border: none;
  border-radius: 0;
  font: inherit;
}

.stuff-card-zoom {
  position: absolute;
  top: 12px;
  right: 12px;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(10, 7, 6, 0.55);
  border: 1px solid rgba(232, 201, 160, 0.35);
  color: #f5efe6;
  font-size: 15px;
  opacity: 0;
  transform: translateY(-4px);
  transition: opacity 0.3s ease, transform 0.3s ease, background 0.3s ease;
  pointer-events: none;
}

.stuff-card:hover .stuff-card-zoom,
.stuff-card-image:focus-visible .stuff-card-zoom {
  opacity: 1;
  transform: translateY(0);
}

/* ---------- lightbox ---------- */
.lightbox-overlay {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 18px;
  padding: 48px 24px;
  background: rgba(6, 4, 3, 0.88);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
  cursor: zoom-out;
}

.lightbox-img {
  max-width: min(900px, 92vw);
  max-height: 82vh;
  border-radius: 12px;
  box-shadow: 0 30px 80px rgba(0, 0, 0, 0.55);
  cursor: default;
  background: #eceff3;
}

.lightbox-caption {
  font-family: 'Bodoni Moda', serif;
  font-style: italic;
  font-size: 14px;
  letter-spacing: 0.04em;
  color: #e8c9a0;
}

.lightbox-close {
  position: absolute;
  top: 24px;
  right: 28px;
  width: 42px;
  height: 42px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  border: 1px solid rgba(232, 201, 160, 0.35);
  background: rgba(245, 239, 230, 0.06);
  color: #f5efe6;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.25s ease, border-color 0.25s ease;
}

.lightbox-close:hover {
  background: rgba(179, 18, 23, 0.35);
  border-color: rgba(179, 18, 23, 0.6);
}

.lightbox-fade-enter-active,
.lightbox-fade-leave-active {
  transition: opacity 0.25s ease;
}

.lightbox-fade-enter-from,
.lightbox-fade-leave-to {
  opacity: 0;
}

@media (max-width: 560px) {
  .lightbox-close {
    top: 16px;
    right: 16px;
    width: 38px;
    height: 38px;
  }
}

.stuff-card-arrow {
  position: absolute;
  top: 28px;
  right: 24px;
  font-size: 18px;
  color: #e8c9a0;
  transition: transform 0.3s ease, color 0.3s ease;
}

.stuff-card:hover .stuff-card-arrow {
  transform: translate(4px, -4px);
  color: #b31217;
}

/* ---------- tech stack chips ---------- */
.stuff-grid--tech {
  grid-template-columns: repeat(3, 1fr);
}

.stuff-chip {
  padding: 22px 24px;
  border-radius: 16px;
  background: rgba(245, 239, 230, 0.04);
  border: 1px solid rgba(232, 201, 160, 0.14);
  display: flex;
  flex-direction: column;
  gap: 6px;
  transition: border-color 0.3s ease, transform 0.3s ease;
}

.stuff-chip:hover {
  border-color: rgba(179, 18, 23, 0.5);
  transform: translateY(-4px);
}

.stuff-chip-name {
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: 17px;
}

.stuff-chip-category {
  font-size: 11px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #cfc6b8;
}

/* ---------- tab switch transition ---------- */
.stuff-fade-enter-active,
.stuff-fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.stuff-fade-enter-from {
  opacity: 0;
  transform: translateY(8px);
}

.stuff-fade-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

/* ---------- contact / CTA ---------- */
.contact {
  position: relative;
  z-index: 1;
  padding: 90px 64px 110px;
  max-width: 1400px;
  margin: 0 auto;
  border-top: 1px solid rgba(245, 239, 230, 0.08);
  text-align: center;
}

.contact-title {
  font-family: 'Bodoni Moda', serif;
  font-weight: 500;
  font-size: clamp(36px, 6vw, 72px);
  line-height: 1.1;
  margin: 20px 0 30px;
}

.contact-title-outline {
  color: transparent;
  -webkit-text-stroke: 1.5px #f5efe6;
}

.contact-email {
  display: inline-block;
  font-family: 'Bodoni Moda', serif;
  font-style: italic;
  font-size: 22px;
  color: #e8c9a0;
  text-decoration: none;
  border-bottom: 1px solid rgba(232, 201, 160, 0.4);
  padding-bottom: 4px;
  margin-bottom: 40px;
}

.contact-links {
  display: flex;
  justify-content: center;
  gap: 28px;
}

.contact-link {
  font-size: 13px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #cfc6b8;
  text-decoration: none;
  transition: color 0.25s ease;
}

.contact-link:hover {
  color: #f5efe6;
}

/* ---------- footer ---------- */
.site-footer {
  position: relative;
  z-index: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 16px;
  padding: 28px 64px;
  max-width: 1400px;
  margin: 0 auto;
  border-top: 1px solid rgba(245, 239, 230, 0.08);
  font-size: 12px;
  letter-spacing: 0.08em;
  color: #cfc6b8;
}

.footer-top {
  color: #cfc6b8;
  text-decoration: none;
  transition: color 0.25s ease;
}

.footer-top:hover {
  color: #e8c9a0;
}

/* ---------- responsive ---------- */
@media (max-width: 900px) {
  .hero,
  .aboutme,
  .skills {
    padding: 40px 28px 70px;
  }

  .hero {
    padding-top: 100px;
  }

  .aboutme-grid {
    grid-template-columns: 1fr;
    gap: 48px;
  }

  .aboutme-photo {
    order: -1;
  }

  .aboutme-lower {
    gap: 32px;
  }

  .aboutme-history {
    grid-template-columns: 1fr;
    gap: 32px;
    margin-top: 32px;
  }

  .aboutme-edu {
    padding-left: 0;
    border-left: none;
    padding-top: 24px;
    border-top: 1px solid rgba(245, 239, 230, 0.08);
    flex-basis: 100%;
  }

  .about {
    grid-template-columns: 1fr;
  }

  .about-photo-frame {
    width: 140px;
  }

  .side-marquee {
    display: none;
  }

  .skills-top {
    flex-direction: column;
    gap: 8px;
  }

  .skills-meta--right {
    text-align: left;
  }

  .projects,
  .contact,
  .site-footer {
    padding-left: 28px;
    padding-right: 28px;
  }

  .stuff-grid,
  .stuff-grid--tech {
    grid-template-columns: repeat(2, 1fr);
  }

  .stuff-tabbar {
    max-width: 100%;
  }

  .site-footer {
    flex-direction: column;
    text-align: center;
  }
}

@media (max-width: 560px) {
  .aboutme-photo-frame {
    width: 190px;
  }

  .aboutme-fact-value {
    font-size: 26px;
  }

  .hero {
    padding-top: 92px;
  }

  .hero-info {
    gap: 16px;
  }

  .hero-title-line {
    font-size: 11px;
    letter-spacing: 0.22em;
  }

  .projects-title {
    font-size: clamp(30px, 8vw, 40px);
  }

  .stuff-grid,
  .stuff-grid--tech {
    grid-template-columns: 1fr;
  }

  .stuff-tab {
    padding: 12px 12px;
    font-size: 10px;
  }
}

@media (max-width: 420px) {
  .hero,
  .aboutme,
  .skills {
    padding-left: 20px;
    padding-right: 20px;
  }

  .projects,
  .contact,
  .site-footer {
    padding-left: 20px;
    padding-right: 20px;
  }

  .hero-title-label {
    font-size: 15px;
  }

  .hero-title-line {
    letter-spacing: 0.14em;
  }

  .aboutme-photo-frame {
    width: 160px;
  }

  .aboutme-facts {
    gap: 24px;
  }

  .orbit-counter {
    width: 68px;
  }

  .orbit-badge {
    width: 40px;
    height: 40px;
  }

  .orbit-name {
    font-size: 10px;
  }

  .orbit-meta {
    font-size: 8px;
  }

  .stuff-tab {
    font-size: 9px;
    padding: 10px 8px;
  }

  .contact-email {
    font-size: 17px;
  }
}
</style>