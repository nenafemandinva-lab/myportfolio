/* =====================================================
   ECOMMERCE VA PORTFOLIO — MAIN JAVASCRIPT
   ===================================================== */

'use strict';

/* ─── INITIALISE AOS ────────────────────────────────── */
document.addEventListener('DOMContentLoaded', () => {
  AOS.init({
    duration: 800,
    easing: 'ease-out-cubic',
    once: true,
    offset: 60,
  });

  initScrollProgress();
  initNavbar();
  initHamburger();
  initParticles();
  initTypedText();
  initCounters();
  initSkillBars();
  initProcessTabs();
  initContactForm();
  initSmoothScroll();
  setFooterYear();
});

/* ─── SCROLL PROGRESS BAR ───────────────────────────── */
function initScrollProgress() {
  const bar = document.getElementById('scroll-progress');
  window.addEventListener('scroll', () => {
    const total = document.documentElement.scrollHeight - window.innerHeight;
    const pct   = (window.scrollY / total) * 100;
    bar.style.width = pct + '%';
  }, { passive: true });
}

/* ─── NAVBAR SCROLL BEHAVIOUR ───────────────────────── */
function initNavbar() {
  const navbar = document.getElementById('navbar');
  let lastScroll = 0;

  window.addEventListener('scroll', () => {
    const current = window.scrollY;
    if (current > 60) {
      navbar.classList.add('scrolled');
    } else {
      navbar.classList.remove('scrolled');
    }
    lastScroll = current;
  }, { passive: true });
}

/* ─── HAMBURGER MENU ────────────────────────────────── */
function initHamburger() {
  const hamburger = document.getElementById('hamburger');
  const navLinks  = document.getElementById('nav-links');

  hamburger.addEventListener('click', () => {
    hamburger.classList.toggle('open');
    navLinks.classList.toggle('open');
    document.body.style.overflow = navLinks.classList.contains('open') ? 'hidden' : '';
  });

  // Close on link click
  navLinks.querySelectorAll('a').forEach(link => {
    link.addEventListener('click', () => {
      hamburger.classList.remove('open');
      navLinks.classList.remove('open');
      document.body.style.overflow = '';
    });
  });
}

/* ─── PARTICLE CANVAS ───────────────────────────────── */
function initParticles() {
  const canvas = document.getElementById('particles-canvas');
  if (!canvas) return;
  const ctx = canvas.getContext('2d');

  let particles = [];
  let animId;

  function resize() {
    canvas.width  = canvas.offsetWidth;
    canvas.height = canvas.offsetHeight;
  }

  class Particle {
    constructor() { this.reset(); }
    reset() {
      this.x    = Math.random() * canvas.width;
      this.y    = Math.random() * canvas.height;
      this.size = Math.random() * 2.5 + 0.5;
      this.speedX = (Math.random() - 0.5) * 0.4;
      this.speedY = (Math.random() - 0.5) * 0.4;
      this.opacity = Math.random() * 0.5 + 0.1;
      this.gold = Math.random() > 0.5;
    }
    update() {
      this.x += this.speedX;
      this.y += this.speedY;
      if (this.x < 0 || this.x > canvas.width ||
          this.y < 0 || this.y > canvas.height) {
        this.reset();
      }
    }
    draw() {
      ctx.save();
      ctx.globalAlpha = this.opacity;
      ctx.fillStyle = this.gold ? '#C9A84C' : '#A07830';
      ctx.beginPath();
      ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
      ctx.fill();
      ctx.restore();
    }
  }

  function init() {
    resize();
    particles = [];
    const count = Math.min(80, Math.floor((canvas.width * canvas.height) / 10000));
    for (let i = 0; i < count; i++) particles.push(new Particle());
  }

  function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    particles.forEach(p => { p.update(); p.draw(); });

    // Draw connections
    for (let i = 0; i < particles.length; i++) {
      for (let j = i + 1; j < particles.length; j++) {
        const dx = particles[i].x - particles[j].x;
        const dy = particles[i].y - particles[j].y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        if (dist < 100) {
          ctx.save();
          ctx.globalAlpha = (1 - dist / 100) * 0.12;
          ctx.strokeStyle = '#C9A84C';
          ctx.lineWidth = 0.5;
          ctx.beginPath();
          ctx.moveTo(particles[i].x, particles[i].y);
          ctx.lineTo(particles[j].x, particles[j].y);
          ctx.stroke();
          ctx.restore();
        }
      }
    }
    animId = requestAnimationFrame(animate);
  }

  init();
  animate();

  let resizeTimer;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(() => { cancelAnimationFrame(animId); init(); animate(); }, 200);
  });
}

/* ─── TYPED TEXT EFFECT ─────────────────────────────── */
function initTypedText() {
  const el = document.getElementById('typed-text');
  if (!el) return;

  const phrases = [
    'Shopify Product Listing',
    'Product Sourcing & Research',
    'Order Fulfillment',
    'Ecommerce Store Management',
    'Dropshipping Operations',
  ];

  let phraseIdx = 0;
  let charIdx   = 0;
  let deleting  = false;
  let delay     = 100;

  function type() {
    const current = phrases[phraseIdx];
    if (!deleting) {
      el.textContent = current.slice(0, ++charIdx);
      if (charIdx === current.length) { deleting = true; delay = 2200; }
      else { delay = 70; }
    } else {
      el.textContent = current.slice(0, --charIdx);
      if (charIdx === 0) {
        deleting  = false;
        phraseIdx = (phraseIdx + 1) % phrases.length;
        delay = 400;
      } else { delay = 45; }
    }
    setTimeout(type, delay);
  }
  setTimeout(type, 900);
}

/* ─── COUNTER ANIMATION ─────────────────────────────── */
function initCounters() {
  const counters = document.querySelectorAll('.stat-num');
  let animated   = false;

  function animateCounters() {
    if (animated) return;
    const hero = document.getElementById('hero');
    if (!hero) return;
    const rect = hero.getBoundingClientRect();
    if (rect.top < window.innerHeight * 0.8) {
      animated = true;
      counters.forEach(counter => {
        const target = parseInt(counter.dataset.target, 10);
        const duration = 1800;
        const start    = performance.now();
        function update(now) {
          const elapsed = now - start;
          const progress = Math.min(elapsed / duration, 1);
          const ease = 1 - Math.pow(1 - progress, 3);
          counter.textContent = Math.round(ease * target).toLocaleString();
          if (progress < 1) requestAnimationFrame(update);
        }
        requestAnimationFrame(update);
      });
    }
  }

  window.addEventListener('scroll', animateCounters, { passive: true });
  animateCounters();
}

/* ─── SKILL BAR ANIMATION ───────────────────────────── */
function initSkillBars() {
  const fills   = document.querySelectorAll('.skill-fill');
  let animated  = false;

  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting && !animated) {
        animated = true;
        fills.forEach(fill => {
          const w = fill.dataset.width;
          setTimeout(() => {
            fill.style.width = w + '%';
          }, 200);
        });
      }
    });
  }, { threshold: 0.3 });

  const skillsSection = document.getElementById('skills');
  if (skillsSection) observer.observe(skillsSection);
}

/* ─── PROCESS TABS ──────────────────────────────────── */
function initProcessTabs() {
  const tabBtns     = document.querySelectorAll('.tab-btn');
  const tabContents = document.querySelectorAll('.tab-content');

  tabBtns.forEach(btn => {
    btn.addEventListener('click', () => {
      const target = btn.dataset.tab;

      tabBtns.forEach(b => b.classList.remove('active'));
      tabContents.forEach(c => c.classList.remove('active'));

      btn.classList.add('active');
      const targetContent = document.getElementById('tab-' + target);
      if (targetContent) {
        targetContent.classList.add('active');
        // Trigger AOS re-scan for freshly visible elements
        AOS.refresh();
      }
    });
  });
}

/* ─── CONTACT FORM ──────────────────────────────────── */
function initContactForm() {
  const form       = document.getElementById('contact-form');
  const successDiv = document.getElementById('form-success');
  const resetBtn   = document.getElementById('reset-form');
  const submitBtn  = document.getElementById('submit-btn');

  if (!form) return;

  form.addEventListener('submit', async (e) => {
    e.preventDefault();

    // Validate
    let valid = true;
    ['name', 'email', 'message'].forEach(id => {
      const el = document.getElementById(id);
      if (!el.value.trim()) {
        el.classList.add('error');
        valid = false;
      } else {
        el.classList.remove('error');
      }
    });

    const emailEl = document.getElementById('email');
    const emailRx = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (emailEl.value && !emailRx.test(emailEl.value)) {
      emailEl.classList.add('error');
      valid = false;
    }

    if (!valid) {
      shakeForm(form);
      return;
    }

    // Show loading
    const btnText    = submitBtn.querySelector('.btn-text');
    const btnLoading = submitBtn.querySelector('.btn-loading');
    btnText.style.display    = 'none';
    btnLoading.style.display = 'inline-flex';
    submitBtn.disabled       = true;

    // Simulate async (replace with real API call if needed)
    await new Promise(resolve => setTimeout(resolve, 1800));

    // Show success
    form.style.display          = 'none';
    successDiv.style.display    = 'block';
    successDiv.style.animation  = 'fadeInUp 0.5s ease';
  });

  if (resetBtn) {
    resetBtn.addEventListener('click', () => {
      form.reset();
      form.style.display       = 'block';
      successDiv.style.display = 'none';
      const btnText    = submitBtn.querySelector('.btn-text');
      const btnLoading = submitBtn.querySelector('.btn-loading');
      btnText.style.display    = 'inline-flex';
      btnLoading.style.display = 'none';
      submitBtn.disabled       = false;
    });
  }

  // Remove error state on input
  form.querySelectorAll('input, textarea').forEach(el => {
    el.addEventListener('input', () => el.classList.remove('error'));
  });
}

function shakeForm(form) {
  form.style.animation = 'none';
  form.style.transform = 'translateX(0)';
  const keyframes = [
    { transform: 'translateX(0)' },
    { transform: 'translateX(-6px)' },
    { transform: 'translateX(6px)' },
    { transform: 'translateX(-4px)' },
    { transform: 'translateX(4px)' },
    { transform: 'translateX(0)' },
  ];
  form.animate(keyframes, { duration: 320, easing: 'ease-in-out' });
}

/* ─── SMOOTH SCROLL ─────────────────────────────────── */
function initSmoothScroll() {
  document.querySelectorAll('a[href^="#"]').forEach(anchor => {
    anchor.addEventListener('click', (e) => {
      const target = document.querySelector(anchor.getAttribute('href'));
      if (target) {
        e.preventDefault();
        const offset = 80;
        const top    = target.getBoundingClientRect().top + window.scrollY - offset;
        window.scrollTo({ top, behavior: 'smooth' });
      }
    });
  });
}

/* ─── FOOTER YEAR ───────────────────────────────────── */
function setFooterYear() {
  const el = document.getElementById('year');
  if (el) el.textContent = new Date().getFullYear();
}

/* ─── CARD TILT EFFECT ──────────────────────────────── */
(function initTilt() {
  const cards = document.querySelectorAll('.service-card, .reason-card');
  cards.forEach(card => {
    card.addEventListener('mousemove', (e) => {
      const rect = card.getBoundingClientRect();
      const x    = e.clientX - rect.left;
      const y    = e.clientY - rect.top;
      const cx   = rect.width  / 2;
      const cy   = rect.height / 2;
      const rotX = ((y - cy) / cy) * -5;
      const rotY = ((x - cx) / cx) *  5;
      card.style.transform = `perspective(800px) rotateX(${rotX}deg) rotateY(${rotY}deg) translateY(-6px)`;
    });
    card.addEventListener('mouseleave', () => {
      card.style.transform = '';
    });
  });
})();

/* ─── NAVBAR ACTIVE LINK ON SCROLL ──────────────────── */
(function initActiveLinks() {
  const sections = document.querySelectorAll('section[id]');
  const navAs    = document.querySelectorAll('.nav-links a[href^="#"]');

  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const id = entry.target.getAttribute('id');
        navAs.forEach(a => {
          a.classList.remove('active-link');
          if (a.getAttribute('href') === '#' + id) {
            a.classList.add('active-link');
          }
        });
      }
    });
  }, { threshold: 0.35, rootMargin: '-80px 0px -60% 0px' });

  sections.forEach(s => observer.observe(s));
})();

/* ─── CSS ACTIVE LINK STYLE ─────────────────────────── */
const style = document.createElement('style');
style.textContent = `
  .nav-links a.active-link { color: var(--gold-dark) !important; }
  .nav-links a.active-link::after { width: 100% !important; }

  @keyframes fadeInUp {
    from { opacity: 0; transform: translateY(20px); }
    to   { opacity: 1; transform: translateY(0);    }
  }
`;
document.head.appendChild(style);
