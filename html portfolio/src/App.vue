<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import cleanSpacesImage from '../cleanspaces.png'
import hrClockImage from '../hr-clock.png'

const menuOpen = ref(false)
const activeSection = ref('home')
const messageSent = ref(false)

let animationFrame
let revealObserver
let sentTimeout
let canvas
let context
let particles = []
let mouse = { x: window.innerWidth / 2, y: window.innerHeight / 2 }

function resizeCanvas() {
  if (!canvas || !context) return
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight
  createParticles()
}

function createParticles() {
  if (!canvas) return
  const amount = Math.floor((canvas.width * canvas.height) / 16000)
  particles = Array.from({ length: amount }, () => ({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    vx: (Math.random() - 0.5) * 0.45,
    vy: (Math.random() - 0.5) * 0.45,
    size: Math.random() * 2 + 0.4,
  }))
}

function drawParticles() {
  if (!canvas || !context) return
  context.clearRect(0, 0, canvas.width, canvas.height)

  for (let i = 0; i < particles.length; i += 1) {
    const particle = particles[i]
    particle.x += particle.vx
    particle.y += particle.vy

    if (particle.x < 0 || particle.x > canvas.width) particle.vx *= -1
    if (particle.y < 0 || particle.y > canvas.height) particle.vy *= -1

    const dx = mouse.x - particle.x
    const dy = mouse.y - particle.y
    if (Math.hypot(dx, dy) < 140) {
      particle.x -= dx * 0.002
      particle.y -= dy * 0.002
    }

    context.beginPath()
    context.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2)
    context.fillStyle = 'rgba(182,255,0,0.55)'
    context.fill()

    for (let j = i + 1; j < particles.length; j += 1) {
      const other = particles[j]
      const distance = Math.hypot(particle.x - other.x, particle.y - other.y)
      if (distance < 120) {
        context.beginPath()
        context.moveTo(particle.x, particle.y)
        context.lineTo(other.x, other.y)
        context.strokeStyle = `rgba(182,255,0,${1 - distance / 120})`
        context.lineWidth = 0.25
        context.stroke()
      }
    }
  }

  animationFrame = requestAnimationFrame(drawParticles)
}

function handleMouseMove(event) {
  mouse.x = event.clientX
  mouse.y = event.clientY

  const cursor = document.querySelector('.cursor')
  const cursorDot = document.querySelector('.cursor-dot')
  if (cursor && cursorDot) {
    cursor.style.left = `${event.clientX}px`
    cursor.style.top = `${event.clientY}px`
    cursorDot.style.left = `${event.clientX}px`
    cursorDot.style.top = `${event.clientY}px`
  }

  const orbitalCard = document.querySelector('.orbital-card')
  if (orbitalCard) {
    const x = (event.clientX / window.innerWidth - 0.5) * 18
    const y = (event.clientY / window.innerHeight - 0.5) * 18
    orbitalCard.style.transform = `rotateX(${8 - y}deg) rotateY(${-12 + x}deg)`
  }
}

function handlePointerOver(event) {
  const target = event.target.closest('a, button, .project-card, .skill-panel, .stat-card')
  if (target && !target.contains(event.relatedTarget)) {
    document.querySelector('.cursor')?.classList.add('active')
  }
}

function handlePointerOut(event) {
  const target = event.target.closest('a, button, .project-card, .skill-panel, .stat-card')
  if (target && !target.contains(event.relatedTarget)) {
    document.querySelector('.cursor')?.classList.remove('active')
  }
}

function handleScroll() {
  const sections = document.querySelectorAll('main section[id]')
  for (const section of sections) {
    if (window.scrollY >= section.offsetTop - 180) activeSection.value = section.id
  }
}

function closeMenu() {
  menuOpen.value = false
}

function tiltCard(event) {
  if (window.matchMedia('(max-width: 760px)').matches) return
  const card = event.currentTarget
  const rect = card.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top
  const rotateX = ((y - rect.height / 2) / (rect.height / 2)) * -4
  const rotateY = ((x - rect.width / 2) / (rect.width / 2)) * 4
  card.style.transform = `perspective(900px) rotateX(${rotateX}deg) rotateY(${rotateY}deg)`
}

function resetCardTilt(event) {
  event.currentTarget.style.transform = ''
}

function submitContact() {
  messageSent.value = true
  window.clearTimeout(sentTimeout)
  sentTimeout = window.setTimeout(() => {
    messageSent.value = false
    document.querySelector('.contact-form')?.reset()
  }, 2000)
}

onMounted(() => {
  canvas = document.querySelector('#bg-canvas')
  context = canvas?.getContext('2d')
  resizeCanvas()
  drawParticles()

  revealObserver = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('show')
        revealObserver.unobserve(entry.target)
      }
    })
  }, { threshold: 0.18 })

  document.querySelectorAll('.reveal').forEach((element) => revealObserver.observe(element))
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('resize', resizeCanvas)
  window.addEventListener('scroll', handleScroll, { passive: true })
  handleScroll()
})

onUnmounted(() => {
  cancelAnimationFrame(animationFrame)
  window.clearTimeout(sentTimeout)
  revealObserver?.disconnect()
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('resize', resizeCanvas)
  window.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <div class="app-shell" @pointerover="handlePointerOver" @pointerout="handlePointerOut">
    <canvas id="bg-canvas" aria-hidden="true"></canvas>
    <div class="cursor" aria-hidden="true"></div>
    <div class="cursor-dot" aria-hidden="true"></div>
    <div class="noise" aria-hidden="true"></div>

    <header class="site-header">
      <a href="#home" class="logo" @click="closeMenu"><span>Mubaarik</span> Davids</a>

      <nav class="nav" :class="{ open: menuOpen }" id="nav" aria-label="Main navigation">
        <a v-for="section in ['home', 'about', 'skills', 'projects', 'contact']"
          :key="section" :href="`#${section}`" :class="{ active: activeSection === section }"
          @click="closeMenu">{{ section.charAt(0).toUpperCase() + section.slice(1) }}</a>
      </nav>

      <button class="menu-btn" :class="{ active: menuOpen }" type="button"
        :aria-expanded="menuOpen" aria-label="Toggle navigation" @click="menuOpen = !menuOpen">
        <span></span><span></span>
      </button>
    </header>

    <main>
      <section class="hero section" id="home">
        <div class="hero-grid">
          <div class="hero-content reveal">
            <p class="eyebrow">Creative Developer / Digital Designer</p>
            <h1>Building <span class="gradient-text">immersive</span> digital experiences.</h1>
            <p class="hero-text">I design and develop futuristic web experiences that feel alive,
              convert attention into action, and help brands stand out online.</p>
            <div class="hero-actions">
              <a href="#projects" class="btn primary">View Projects</a>
              <a href="#contact" class="btn secondary">Let’s Talk</a>
            </div>
          </div>

          <div class="hero-visual reveal">
            <div class="orbital-card">
              <div class="ring ring-1"></div><div class="ring ring-2"></div><div class="ring ring-3"></div>
              <div class="avatar-core"><span></span></div>
              <div class="floating-pill pill-1">HTML</div><div class="floating-pill pill-2">CSS</div>
              <div class="floating-pill pill-3">JS</div><div class="floating-pill pill-4">UI/UX</div>
            </div>
          </div>
        </div>
        <div class="scroll-indicator"><span></span>Scroll</div>
      </section>

      <section class="about section" id="about">
        <div class="section-heading reveal"><p class="eyebrow">About Me</p>
          <h2>Not just a developer, a digital experience builder.</h2></div>
        <div class="about-layout">
          <div class="about-card reveal">
            <h3>Who I Am</h3>
            <p>Hi, I'm <b>Mubaarik Davids</b>, an aspiring developer currently beginning my journey at Life Choices Academy.
              I'm passionate about learning new technologies, solving problems, and building projects that help me grow as a developer.
              Every day is an opportunity to improve my skills and gain hands-on experience in the world of software development.
              My long-term goal is to become a professional full-stack developer, creating meaningful and user-friendly applications while
              continuously learning and adapting in this ever-evolving industry. I'm excited about the journey ahead and the opportunities it will bring.</p>
            <br />
            <p>As I continue to grow in this field, I'm eager to take on new challenges, collaborate with others, and turn ideas into
              real-world solutions. I believe that dedication, curiosity, and continuous learning are key to success, and I'm committed
              to making the most of every opportunity that comes my way.</p>
          </div>
          <div class="about-stack reveal">
            <div class="stat-card"><h4>20+</h4><p>Interfaces Designed</p></div>
            <div class="stat-card"><h4>100%</h4><p>Responsive Builds</p></div>
            <div class="stat-card"><h4>Clean</h4><p>Code Structure</p></div>
          </div>
          <div class="about-timeline reveal">
            <div class="timeline-item"><span>01</span><div><h4>Strategy</h4><p>Understanding the brand, goals, audience, and experience.</p></div></div>
            <div class="timeline-item"><span>02</span><div><h4>Design</h4><p>Building visual direction, layout, motion, and structure.</p></div></div>
            <div class="timeline-item"><span>03</span><div><h4>Development</h4><p>Creating fast, responsive, and interactive websites.</p></div></div>
          </div>
        </div>
      </section>

      <section class="skills section" id="skills">
        <div class="section-heading reveal"><p class="eyebrow">Skills</p>
          <h2>Tools and abilities that power the experience.</h2></div>
        <div class="skills-layout">
          <div class="skills-orbit reveal"><div class="orbit-center">CORE</div>
            <div class="skill-node node-1">HTML</div><div class="skill-node node-2">CSS</div>
            <div class="skill-node node-3">JS</div><div class="skill-node node-4">UI</div>
            <div class="skill-node node-5">UX</div><div class="skill-node node-6">SEO</div>
          </div>
          <div class="skill-panels reveal">
            <div class="skill-panel"><div><h3>Frontend Development</h3><p>Semantic HTML, responsive CSS, JavaScript, animations, and browser-friendly performance.</p></div><span>95%</span></div>
            <div class="skill-panel"><div><h3>Visual Design</h3><p>Clean layouts, strong spacing, visual hierarchy, modern effects, and premium interfaces.</p></div><span>90%</span></div>
            <div class="skill-panel"><div><h3>Interactive Experiences</h3><p>Scroll effects, custom cursors, animated cards, glassmorphism, and smooth micro-interactions.</p></div><span>88%</span></div>
            <div class="skill-panel"><div><h3>Responsive Design</h3><p>Fluid layouts that adapt beautifully across desktop, tablet, and mobile devices.</p></div><span>96%</span></div>
          </div>
        </div>
      </section>

      <section class="projects section" id="projects">
        <div class="section-heading reveal"><p class="eyebrow">Projects</p>
          <h2>Selected work crafted for impact.</h2></div>
        <div class="project-grid">
          <article class="project-card reveal" @mousemove="tiltCard" @mouseleave="resetCardTilt">
            <a class="project-image image-1" href="https://cleanspaces.onrender.com" target="_blank" rel="noopener noreferrer" aria-label="Open the CleanSpaces e-commerce site in a new tab">
              <img :src="cleanSpacesImage" alt="CleanSpaces community cleanup website" />
            </a>
            <div class="project-content"><p>Website</p><h3>E-commerce Site</h3><span><a href="https://github.com/ihtishaamj63-web/CleanSpaces.git" target="_blank" rel="noopener noreferrer">GitHub repository</a></span></div>
          </article>
          <article class="project-card reveal" @mousemove="tiltCard" @mouseleave="resetCardTilt">
            <a class="project-image image-2" href="https://modern-tech-solutions00.netlify.app/dashboard.html" target="_blank" rel="noopener noreferrer" aria-label="Open the live HR Clock-In System dashboard in a new tab">
              <img :src="hrClockImage" alt="ModernTech HR company dashboard" />
            </a>
            <div class="project-content"><p>Dashboard</p><h3>HR Clock-In System</h3><span><a href="https://github.com/purritygwabeni43-tech/Modern-tech-back-end.git" target="_blank" rel="noopener noreferrer">GitHub repository</a></span></div>
          </article>
          <article class="project-card reveal" @mousemove="tiltCard" @mouseleave="resetCardTilt">
            <div class="project-image image-3"></div>
            <div class="project-content"><p>Brand Website</p><h3>Genealogy Company Site</h3><span>Story-driven website for heritage and family research.</span></div>
          </article>
          <article class="project-card reveal" @mousemove="tiltCard" @mouseleave="resetCardTilt">
            <div class="project-image image-4"></div>
            <div class="project-content"><p>Business Platform</p><h3>Client Management System</h3><span>Modern system for managing clients, projects, and workflow.</span></div>
          </article>
        </div>
      </section>

      <section class="contact section" id="contact">
        <div class="contact-card reveal" @mousemove="tiltCard" @mouseleave="resetCardTilt">
          <div><p class="eyebrow">Contact</p><h2>Have a bold idea? Let’s build something insane.</h2>
            <p>Send a message and let’s turn your concept into a premium digital experience.</p>
            <div class="contact-links" aria-label="Social profiles">
              <a class="btn secondary" href="https://www.linkedin.com/in/baarikthegoat" target="_blank" rel="noopener noreferrer">LinkedIn</a>
              <a class="btn secondary" href="https://github.com/mubaarikdavids" target="_blank" rel="noopener noreferrer">GitHub</a>
            </div>
          </div>
          <form class="contact-form" @submit.prevent="submitContact">
            <div class="input-group"><input type="text" required /><label>Your Name</label></div>
            <div class="input-group"><input type="email" required /><label>Email Address</label></div>
            <div class="input-group"><textarea required></textarea><label>Your Message</label></div>
            <button type="submit" class="btn primary">{{ messageSent ? 'Message Sent' : 'Send Message' }}</button>
          </form>
        </div>
      </section>
    </main>

    <footer><p>© 2026 Mubaarik Davids. All rights reserved</p></footer>
  </div>
</template>
