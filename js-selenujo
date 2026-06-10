/**
 * js-selenujo.js
 * Selenujo RPG — comportements dynamiques
 * Blank Theme v3.5 / ForumActif ModernBB
 *
 * Fonctionnalités :
 *  1. Collapse des catégories (▲/▼)
 *  2. Carousel "En ce moment" (news)
 *  3. Phase lunaire automatique
 *  4. Layout spécial Admin (inject template)
 *  5. Layout spécial Hors RP (inject template)
 *  6. Notiffi panel toggle
 *  7. Nettoyage textes FA (QEEL)
 */

(function () {
  'use strict';

  /* ══════════════════════════════════════════════
     1. COLLAPSE DES CATÉGORIES
     ══════════════════════════════════════════════ */
  function initCollapse() {
    var triggers = document.querySelectorAll('[data-collapse-trigger]');
    triggers.forEach(function (trigger) {
      // Restaurer l'état depuis localStorage
      var category = trigger.closest('[data-sel-category]');
      if (!category) return;
      var id = category.id || category.getAttribute('data-sel-category') || Math.random();
      var storageKey = 'sel-cat-' + id;
      var collapsed = localStorage.getItem(storageKey) === 'collapsed';

      if (collapsed) {
        category.classList.add('collapsed');
        trigger.classList.add('collapsed');
      }

      trigger.addEventListener('click', function () {
        var isCollapsed = category.classList.toggle('collapsed');
        trigger.classList.toggle('collapsed', isCollapsed);
        localStorage.setItem(storageKey, isCollapsed ? 'collapsed' : 'open');
      });

      // Accessibilité clavier
      trigger.addEventListener('keydown', function (e) {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          trigger.click();
        }
      });
    });
  }

  /* ══════════════════════════════════════════════
     2. CAROUSEL NEWS "EN CE MOMENT"
     ══════════════════════════════════════════════ */
  function initCarousel() {
    var carousel = document.getElementById('sel-news-carousel');
    if (!carousel) return;

    var track    = document.getElementById('sel-news-track');
    var slides   = track ? track.querySelectorAll('.sel-news-slide') : [];
    var dots     = document.querySelectorAll('.sel-news-dot');
    var prevBtn  = document.getElementById('sel-news-prev');
    var nextBtn  = document.getElementById('sel-news-next');

    if (!slides.length) return;

    var current  = 0;
    var total    = slides.length;
    var autoTimer = null;
    var AUTO_DELAY = 5000; // 5 secondes

    function goTo(index) {
      current = (index + total) % total;
      if (track) track.style.transform = 'translateX(-' + (current * 100) + '%)';
      dots.forEach(function (d, i) {
        d.classList.toggle('active', i === current);
      });
    }

    function startAuto() {
      stopAuto();
      autoTimer = setInterval(function () { goTo(current + 1); }, AUTO_DELAY);
    }

    function stopAuto() {
      if (autoTimer) { clearInterval(autoTimer); autoTimer = null; }
    }

    if (prevBtn) prevBtn.addEventListener('click', function () { goTo(current - 1); startAuto(); });
    if (nextBtn) nextBtn.addEventListener('click', function () { goTo(current + 1); startAuto(); });

    dots.forEach(function (dot) {
      dot.addEventListener('click', function () {
        goTo(parseInt(dot.getAttribute('data-slide'), 10));
        startAuto();
      });
    });

    // Pause au survol
    carousel.addEventListener('mouseenter', stopAuto);
    carousel.addEventListener('mouseleave', startAuto);

    // Swipe tactile
    var touchStartX = 0;
    carousel.addEventListener('touchstart', function (e) {
      touchStartX = e.changedTouches[0].clientX;
    }, { passive: true });
    carousel.addEventListener('touchend', function (e) {
      var diff = touchStartX - e.changedTouches[0].clientX;
      if (Math.abs(diff) > 40) {
        goTo(diff > 0 ? current + 1 : current - 1);
        startAuto();
      }
    }, { passive: true });

    goTo(0);
    startAuto();
  }

  /* ══════════════════════════════════════════════
     3. PHASE LUNAIRE AUTOMATIQUE
     Calcul basé sur un cycle synodique de 29.53059 jours.
     Date de référence : nouvelle lune connue = 2024-01-11
     ══════════════════════════════════════════════ */
  function initMoonPhase() {
    var moonIcon  = document.getElementById('sel-moon-icon');
    var moonLabel = document.getElementById('sel-moon-label');
    var moonSub   = document.getElementById('sel-moon-sub');
    if (!moonIcon || !moonLabel) return;

    var KNOWN_NEW_MOON = new Date('2024-01-11T11:57:00Z');
    var SYNODIC = 29.53059;

    var now      = new Date();
    var diffDays = (now - KNOWN_NEW_MOON) / (1000 * 60 * 60 * 24);
    var age      = ((diffDays % SYNODIC) + SYNODIC) % SYNODIC; // 0 = nouvelle lune

    var phases = [
      { max: 1.85,  icon: '🌑', label: 'Nouvelle lune',         sub: 'Nuit complète · Obscurité totale' },
      { max: 7.38,  icon: '🌒', label: 'Premier croissant',      sub: 'Nuit · Faible luminosité' },
      { max: 9.22,  icon: '🌓', label: 'Premier quartier',       sub: 'Nuit · Demi-lumière' },
      { max: 14.77, icon: '🌔', label: 'Lune gibbeuse croissante', sub: 'Nuit · Bonne visibilité' },
      { max: 16.61, icon: '🌕', label: 'Pleine lune',            sub: 'Nuit · Visibilité maximale' },
      { max: 22.15, icon: '🌖', label: 'Lune gibbeuse décroissante', sub: 'Nuit · Bonne visibilité' },
      { max: 23.99, icon: '🌗', label: 'Dernier quartier',       sub: 'Nuit · Demi-lumière' },
      { max: 29.53, icon: '🌘', label: 'Dernier croissant',      sub: 'Nuit · Faible luminosité' }
    ];

    var phase = phases[phases.length - 1];
    for (var i = 0; i < phases.length; i++) {
      if (age < phases[i].max) { phase = phases[i]; break; }
    }

    moonIcon.textContent  = phase.icon;
    moonLabel.textContent = phase.label;
    if (moonSub) moonSub.textContent = phase.sub;
  }

  /* ══════════════════════════════════════════════
     4. LAYOUT ADMIN — injection du template
     Détecte la catégorie dont le titre contient
     "Administration" et injecte tpl-admin-inner
     ══════════════════════════════════════════════ */
  function buildAdminLayout() {
    var tpl = document.getElementById('tpl-admin-inner');
    if (!tpl) return;

    var categories = document.querySelectorAll('[data-sel-category]');
    categories.forEach(function (cat) {
      var title = cat.querySelector('.cate_title');
      if (!title) return;
      if (title.textContent.indexOf('Administration') === -1) return;

      cat.classList.add('sel-admin');

      var forumsContainer = cat.querySelector('.forums');
      if (!forumsContainer) return;

      // Sauvegarder le contenu FA original (liens fonctionnels)
      var existing = forumsContainer.innerHTML;
      forumsContainer.innerHTML = '';

      // Cloner le contenu du div masqué (pas de .content comme sur <template>)
      var clone = tpl.cloneNode(true);
      clone.removeAttribute('id');
      clone.style.display = '';
      forumsContainer.appendChild(clone);

      // Réinjecter les forums FA originaux masqués
      var hiddenContainer = document.createElement('div');
      hiddenContainer.style.display = 'none';
      hiddenContainer.innerHTML = existing;
      forumsContainer.appendChild(hiddenContainer);
    });
  }

  /* ══════════════════════════════════════════════
     5. LAYOUT HORS RP — injection du template
     ══════════════════════════════════════════════ */
  function buildHorsRPLayout() {
    var tpl = document.getElementById('tpl-horsrp-inner');
    if (!tpl) return;

    var categories = document.querySelectorAll('[data-sel-category]');
    categories.forEach(function (cat) {
      var title = cat.querySelector('.cate_title');
      if (!title) return;
      var titleText = title.textContent.trim();
      if (titleText.indexOf('Hors RP') === -1 && titleText.indexOf('Hors-RP') === -1) return;

      cat.classList.add('sel-horsrp');

      var forumsContainer = cat.querySelector('.forums');
      if (!forumsContainer) return;

      var existing = forumsContainer.innerHTML;
      forumsContainer.innerHTML = '';

      var clone = tpl.cloneNode(true);
      clone.removeAttribute('id');
      clone.style.display = '';
      forumsContainer.appendChild(clone);

      var hiddenContainer = document.createElement('div');
      hiddenContainer.style.display = 'none';
      hiddenContainer.innerHTML = existing;
      forumsContainer.appendChild(hiddenContainer);
    });
  }

  /* ══════════════════════════════════════════════
     6. NOTIFFI PANEL TOGGLE
     Ouvre/ferme le panneau sans navigation
     ══════════════════════════════════════════════ */
  function initNotiffiToggle() {
    var btn   = document.getElementById('notiffi_button');
    var panel = document.getElementById('notiffi_panel');
    if (!btn || !panel) return;

    btn.addEventListener('click', function (e) {
      e.stopPropagation();
      panel.classList.toggle('open');
    });

    // Fermer en cliquant ailleurs
    document.addEventListener('click', function (e) {
      if (!panel.contains(e.target) && e.target !== btn) {
        panel.classList.remove('open');
      }
    });
  }

  /* ══════════════════════════════════════════════
     7. NETTOYAGE TEXTES FA (QEEL)
     Déjà géré en inline dans index_body.html,
     mais on refait ici pour sécurité si jQuery absent
     ══════════════════════════════════════════════ */
  function cleanQEELTexts() {
    function replaceText(id, search, replace) {
      var el = document.getElementById(id);
      if (!el) return;
      el.innerHTML = el.innerHTML.replace(search, replace || '');
    }
    replaceText('last_user', "L'utilisateur enregistré le plus récent est");
    replaceText('qeel_posts', "Nos membres ont posté un total de");
    replaceText('qeel_members', "Nous avons");
    replaceText('total_users', "Il y a en tout");
    replaceText('total_users', "utilisateur en ligne", "connecté");
    replaceText('total_users', "utilisateurs en ligne", "connectés");
    replaceText('online_users', "Utilisateurs enregistrés", "En ligne");
    replaceText('last_connected', "Membres connectés au cours des 24 dernières heures :", "Connectés récemment");
  }

  /* ══════════════════════════════════════════════
     8. NAVIGATION CLAVIER CAROUSEL
     ══════════════════════════════════════════════ */
  function initCarouselKeyboard() {
    document.addEventListener('keydown', function (e) {
      var carousel = document.getElementById('sel-news-carousel');
      if (!carousel) return;
      // Seulement si le focus est dans le carousel ou ses boutons
      if (document.activeElement && carousel.contains(document.activeElement)) {
        if (e.key === 'ArrowLeft') document.getElementById('sel-news-prev') && document.getElementById('sel-news-prev').click();
        if (e.key === 'ArrowRight') document.getElementById('sel-news-next') && document.getElementById('sel-news-next').click();
      }
    });
  }

  /* ══════════════════════════════════════════════
     9. RESPONSIVE IMAGE DANS FORUM-IMG-WRAP
     Ajuste la hauteur de l'image selon le contenu
     ══════════════════════════════════════════════ */
  function equalizeForumImageHeights() {
    var rows = document.querySelectorAll('.forum.row');
    rows.forEach(function (row) {
      var imgWrap = row.querySelector('.forum-img-wrap');
      var content = row.querySelector('.forum_content');
      if (!imgWrap || !content) return;
      // L'image suit naturellement la hauteur de la grille CSS (align-self: stretch)
      // On s'assure juste que l'img a bien height:100%
      var img = imgWrap.querySelector('img');
      if (img) img.style.height = '100%';
    });
  }

  /* ══════════════════════════════════════════════
     INIT — DOMContentLoaded
     ══════════════════════════════════════════════ */
  function init() {
    initCollapse();
    initCarousel();
    initCarouselKeyboard();
    initMoonPhase();
    buildAdminLayout();
    buildHorsRPLayout();
    initNotiffiToggle();
    cleanQEELTexts();
    equalizeForumImageHeights();
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }

})();
