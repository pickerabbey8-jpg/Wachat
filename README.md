<html lang="en"><head>
<!-- Playables SDK -->
<script>// Playables SDK v1.0.0
// Game lifecycle bridge: rAF-based game-ready detection + event communication
(function() {
  'use strict';

  // Idempotency: skip if already initialized (e.g., server-side injection
  // followed by client-side inject-javascript via the Bloks webview component).
  if (window.playablesSDK) return;

  var HANDLER_NAME = 'playablesGameEventHandler';
  var ANDROID_BRIDGE_NAME = '_MetaPlayablesBridge';
  var RAF_FRAME_THRESHOLD = 3;

  var gameReadySent = false;
  var firstInteractionSent = false;
  var errorSent = false;
  var frameCount = 0;
  var originalRAF = window.requestAnimationFrame;

  // --- Transport Layer ---

  function hasIOSBridge() {
    return !!(window.webkit &&
              window.webkit.messageHandlers &&
              window.webkit.messageHandlers[HANDLER_NAME]);
  }

  function hasAndroidBridge() {
    return !!(window[ANDROID_BRIDGE_NAME] &&
              typeof window[ANDROID_BRIDGE_NAME].postEvent === 'function');
  }

  function isInIframe() {
    return !!(window.parent && window.parent !== window);
  }

  function sendEvent(eventName, payload) {
    var message = {
      type: eventName,
      payload: payload || {},
      timestamp: Date.now()
    };

    if (hasIOSBridge()) {
      try {
        window.webkit.messageHandlers[HANDLER_NAME].postMessage(message);
      } catch (e) { /* ignore */ }
      return;
    }

    if (hasAndroidBridge()) {
    try {
      var p = payload || {};
      p.__secureToken = window.__fbAndroidBridgeAuthToken || '';
      p.timestamp = message.timestamp;
      window[ANDROID_BRIDGE_NAME].postEvent(
        eventName,
        JSON.stringify(p)
      );
    } catch (e) { /* ignore */ }
    return;
  }

    if (isInIframe()) {
      try {
        window.parent.postMessage(message, '*');
      } catch (e) { /* ignore */ }
      return;
    }
  }

  // --- rAF Game-Ready Detection ---

  function onFrame() {
    if (gameReadySent) return;

    frameCount++;
    if (frameCount >= RAF_FRAME_THRESHOLD) {
      gameReadySent = true;
      sendEvent('game_ready', {
        frame_count: frameCount,
        detected_at: Date.now()
      });
      return;
    }

    originalRAF.call(window, onFrame);
  }

  if (originalRAF) {
    window.requestAnimationFrame = function(callback) {
      if (!gameReadySent) {
        return originalRAF.call(window, function(timestamp) {
          frameCount++;
          if (frameCount >= RAF_FRAME_THRESHOLD && !gameReadySent) {
            gameReadySent = true;
            sendEvent('game_ready', {
              frame_count: frameCount,
              detected_at: Date.now()
            });
          }
          callback(timestamp);
        });
      }
      return originalRAF.call(window, callback);
    };
  }

  // --- First User Interaction Detection ---

  function setupFirstInteractionDetection() {
    var events = ['touchstart', 'mousedown', 'keydown'];

    function onFirstInteraction() {
      if (firstInteractionSent) return;
      firstInteractionSent = true;
      sendEvent('user_interaction_start', null);

      for (var i = 0; i < events.length; i++) {
        document.removeEventListener(events[i], onFirstInteraction, true);
      }
    }

    for (var i = 0; i < events.length; i++) {
      document.addEventListener(events[i], onFirstInteraction, true);
    }
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', setupFirstInteractionDetection);
  } else {
    setupFirstInteractionDetection();
  }

  // --- Auto Error Capture ---

  window.addEventListener('error', function(event) {
    if (errorSent) return;
    errorSent = true;
    sendEvent('error', {
      message: event.message || 'Unknown error',
      source: event.filename || '',
      lineno: event.lineno || 0,
      colno: event.colno || 0,
      auto_captured: true
    });
  });

  window.addEventListener('unhandledrejection', function(event) {
    if (errorSent) return;
    errorSent = true;
    var reason = event.reason;
    sendEvent('error', {
      message: (reason instanceof Error) ? reason.message : String(reason),
      type: 'unhandled_promise_rejection',
      auto_captured: true
    });
  });

  // --- Public API ---

  window.playablesSDK = {
    complete: function(score) {
      sendEvent('game_ended', {
        score: score,
        completed: true
      });
    },

    error: function(message) {
      if (errorSent) return;
      errorSent = true;
      sendEvent('error', {
        message: message || 'Unknown error',
        auto_captured: false
      });
    },

    sendEvent: function(eventName, payload) {
      if (!eventName || typeof eventName !== 'string') return;
      sendEvent(eventName, payload);
    }
  };

  // Kick off rAF detection in case no game code calls rAF immediately
  if (originalRAF) {
    originalRAF.call(window, onFrame);
  }
})();</script>
<!-- PLAYABLE_TOUCH_PATCH_V1 --><script>
(function() {
  if (window.__playableTouchPatchInstalled) return;
  window.__playableTouchPatchInstalled = true;
  var origAdd = EventTarget.prototype.addEventListener;
  var blockedTypes = { touchstart: 1, touchmove: 1, wheel: 1 };
  EventTarget.prototype.addEventListener = function(type, listener, options) {
    if (blockedTypes[type]) {
      if (options === undefined || options === null) {
        options = { passive: true };
      } else if (typeof options === 'boolean') {
        options = { capture: options, passive: true };
      } else {
        options = Object.assign({}, options, { passive: true });
      }
    }
    return origAdd.call(this, type, listener, options);
  };
})();
</script><script>window.Intl=window.Intl||{};Intl.t=function(s){return(Intl._locale&&Intl._locale[s])||s;};</script>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>wachat</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
:root {
  --blue: #1877f2;
  --bg: #f0f2f5;
  --card: #ffffff;
  --text: #050505;
  --muted: #65676b;
  --border: #e4e6eb;
  --green: #31a24c;
  --green-bg: #e7f3ff;
}
html, body { height: 100%; }
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  background: var(--bg);
  color: var(--text);
  overscroll-behavior-y: contain;
}
.app { max-width: 500px; margin: 0 auto; min-height: 100vh; background: var(--bg); position: relative; }

/* Header */
header {
  position: fixed; top: 0; left: 0; right: 0; height: 56px; background: var(--blue);
  display: flex; align-items: center; justify-content: space-between; padding: 0 16px;
  z-index: 100; box-shadow: 0 1px 2px rgba(0,0,0,.1);
}
.header-inner { max-width: 500px; width: 100%; margin: 0 auto; display: flex; align-items: center; justify-content: space-between; }
.logo { color: #fff; font-size: 28px; font-weight: 700; letter-spacing: -0.5px; text-transform: lowercase; }
.header-icon { width: 28px; height: 28px; color: #fff; opacity: .95; }

/* Main */
main { padding-top: 56px; padding-bottom: 72px; min-height: 100vh; }
.tab { display: none; }
.tab.active { display: block; }

/* Stories */
.stories-wrap { background: var(--card); padding: 12px 0 10px; margin-bottom: 8px; border-bottom: 1px solid var(--border); }
.stories { display: flex; gap: 14px; overflow-x: auto; padding: 0 12px; scrollbar-width: none; }
.stories::-webkit-scrollbar { display: none; }
.story-item { flex: 0 0 72px; text-align: center; cursor: pointer; }
.story-ring {
  width: 68px; height: 68px; border-radius: 50%; padding: 3px;
  background: var(--border); display: flex; align-items: center; justify-content: center;
  margin: 0 auto 6px; transition: transform .15s;
}
.story-item:active .story-ring { transform: scale(.96); }
.story-ring.active { background: var(--blue); }
.story-ring.add { background: transparent; border: 2px dashed #ccd0d5; }
.story-avatar {
  width: 100%; height: 100%; border-radius: 50%; background: var(--card);
  display: flex; align-items: center; justify-content: center; font-weight: 600;
  color: var(--blue); font-size: 24px; overflow: hidden; position: relative;
}
.story-avatar img { width: 100%; height: 100%; object-fit: cover; }
.story-avatar.add-inner { background: #fff; color: var(--blue); }
.story-name { font-size: 12px; color: var(--muted); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

/* Feed */
.feed { padding: 0 0 16px; }
.empty-state { text-align: center; color: #8a8d91; padding: 80px 20px; font-size: 15px; }
.post-card {
  background: var(--card); border-radius: 12px; margin: 8px 12px; padding: 12px;
  box-shadow: 0 1px 2px rgba(0,0,0,.08);
}
.post-head { display: flex; gap: 10px; align-items: center; }
.post-avatar {
  width: 40px; height: 40px; border-radius: 50%; background: var(--blue);
  color: #fff; display: flex; align-items: center; justify-content: center;
  font-weight: 600; font-size: 16px; flex-shrink: 0; overflow: hidden;
}
.post-avatar img { width: 100%; height: 100%; object-fit: cover; }
.post-meta { flex: 1; min-width: 0; }
.post-name { font-weight: 600; font-size: 15px; line-height: 1.2; }
.post-time { color: var(--muted); font-size: 13px; margin-top: 2px; }
.post-text { margin: 12px 0 10px; font-size: 15px; line-height: 1.4; white-space: pre-wrap; word-break: break-word; }
.post-divider { height: 1px; background: var(--border); margin: 8px -12px; }
.post-actions { display: flex; justify-content: space-between; align-items: center; padding: 4px 2px; }
.action-btn {
  display: flex; align-items: center; gap: 6px; background: none; border: none;
  color: var(--muted); font-size: 14px; font-weight: 500; padding: 6px 8px;
  border-radius: 6px; cursor: pointer;
}
.action-btn:active { background: #f2f2f2; }
.action-btn.liked { color: var(--blue); }
.action-btn svg { width: 18px; height: 18px; }
.comments-count { color: var(--muted); font-size: 13px; padding-right: 6px; }
.comment-bar { display: flex; align-items: center; gap: 8px; margin-top: 8px; }
.comment-avatar {
  width: 28px; height: 28px; border-radius: 50%; background: #e4e6eb;
  display: flex; align-items: center; justify-content: center; font-size: 12px;
  font-weight: 600; color: var(--muted); flex-shrink: 0; overflow: hidden;
}
.comment-avatar img { width: 100%; height: 100%; object-fit: cover; }
.comment-input {
  flex: 1; background: #f0f2f5; border: none; border-radius: 18px;
  padding: 8px 12px; font-size: 14px; outline: none;
}
.comment-send { color: var(--blue); font-weight: 600; background: none; border: none; padding: 0 8px; font-size: 14px; cursor: pointer; }

/* Post Tab */
.post-composer { background: var(--card); border-radius: 12px; margin: 12px; padding: 16px; box-shadow: 0 1px 2px rgba(0,0,0,.08); }
.post-composer textarea {
  width: 100%; min-height: 160px; border: none; outline: none; resize: none;
  font-size: 20px; font-family: inherit; line-height: 1.4; color: var(--text);
}
.post-composer textarea::placeholder { color: #8a8d91; }
.post-btn {
  position: fixed; bottom: 84px; left: 50%; transform: translateX(-50%);
  width: calc(100% - 32px); max-width: 468px; height: 48px;
  background: var(--blue); color: #fff; border: none; border-radius: 24px;
  font-size: 16px; font-weight: 600; cursor: pointer; box-shadow: 0 2px 8px rgba(24,119,242,.3);
  transition: opacity .2s, transform .1s;
}
.post-btn:disabled { opacity: .5; cursor: not-allowed; }
.post-btn:active { transform: translateX(-50%) scale(.98); }

/* Chat Tab */
.placeholder { text-align: center; color: var(--muted); padding: 100px 20px; font-size: 15px; }

/* Me Tab */
.me-card {
  background: var(--card); border-radius: 12px; margin: 16px 12px; padding: 28px 24px;
  text-align: center; box-shadow: 0 1px 2px rgba(0,0,0,.08);
}
.me-card h3 { font-size: 18px; margin-bottom: 6px; }
.me-card p { color: var(--muted); font-size: 14px; margin-bottom: 18px; }
#telegram-login { display: flex; justify-content: center; min-height: 44px; }
.profile-avatar {
  width: 84px; height: 84px; border-radius: 50%; margin: 0 auto 14px;
  background: var(--blue); color: #fff; display: flex; align-items: center;
  justify-content: center; font-size: 32px; font-weight: 600; overflow: hidden;
}
.profile-avatar img { width: 100%; height: 100%; object-fit: cover; }
.profile-name { font-size: 22px; font-weight: 700; }
.profile-username { color: var(--muted); margin-top: 2px; font-size: 15px; }
.green-badge {
  display: inline-flex; align-items: center; gap: 6px; margin-top: 12px;
  background: #e4f7e9; color: #0a7d27; font-size: 13px; font-weight: 500;
  padding: 6px 12px; border-radius: 16px;
}
.logout-btn {
  margin-top: 20px; width: 100%; height: 44px; border: none; border-radius: 8px;
  background: #f0f2f5; font-weight: 600; font-size: 15px; cursor: pointer;
}
.logout-btn:active { background: #e4e6eb; }

/* Bottom Nav */
.bottom-nav {
  position: fixed; bottom: 0; left: 0; right: 0; height: 60px; background: #fff;
  border-top: 1px solid var(--border); display: flex; z-index: 90;
}
.nav-inner { max-width: 500px; width: 100%; margin: 0 auto; display: flex; }
.nav-btn {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  gap: 3px; background: none; border: none; color: var(--muted); font-size: 11px;
  cursor: pointer; padding: 6px 0;
}
.nav-btn svg { width: 26px; height: 26px; }
.nav-btn.active { color: var(--blue); }

/* Story Viewer */
.story-viewer {
  position: fixed; inset: 0; background: #000; z-index: 200;
  display: none; flex-direction: column; color: #fff;
}
.story-viewer.active { display: flex; }
.story-progress { position: absolute; top: 0; left: 0; right: 0; height: 3px; background: rgba(255,255,255,.3); z-index: 2; }
.story-progress-bar { height: 100%; width: 0; background: #fff; animation: prog 5s linear forwards; }
@keyframes prog { to { width: 100%; } }
.story-header {
  position: absolute; top: 0; left: 0; right: 0; padding: 14px 16px;
  display: flex; align-items: center; gap: 10px; background: linear-gradient(to bottom, rgba(0,0,0,.5), transparent);
  z-index: 2; padding-top: 20px;
}
.sv-avatar { width: 36px; height: 36px; border-radius: 50%; background: var(--blue); display: flex; align-items: center; justify-content: center; font-weight: 600; overflow: hidden; }
.sv-avatar img { width: 100%; height: 100%; object-fit: cover; }
.sv-meta { flex: 1; }
.sv-name { font-weight: 600; font-size: 14px; }
.sv-time { font-size: 12px; opacity: .8; }
.sv-close { background: none; border: none; color: #fff; font-size: 26px; line-height: 1; padding: 4px 8px; cursor: pointer; }
.story-content { flex: 1; display: flex; align-items: center; justify-content: center; padding: 40px 24px; }
.story-text { font-size: 28px; line-height: 1.35; text-align: center; word-break: break-word; max-width: 90%; font-weight: 500; }

@media (min-width: 501px) {
  header { box-shadow: 0 1px 0 rgba(0,0,0,.1); }
  .post-btn { bottom: 84px; }
}
</style>
<script>
window.onTelegramAuth = function(user) {
  const u = {
    id: user.id,
    name: `${user.first_name} ${user.last_name || ''}`.trim(),
    first_name: user.first_name,
    last_name: user.last_name || '',
    username: user.username || `user${user.id}`,
    photo_url: user.photo_url || ''
  };
  localStorage.setItem('wachat_user', JSON.stringify(u));
  window.dispatchEvent(new CustomEvent('wachat-login', { detail: u }));
};
</script>
</head>
<body>
<div class="app">
  <header>
    <div class="header-inner">
      <div class="logo">wachat</div>
      <div class="header-icon">
        <svg viewBox="0 0 24 24" fill="white"><path d="M12 3c-4.97 0-9 3.58-9 8 0 1.73.62 3.34 1.69 4.64L3 20l4.7-1.57c1.07.43 2.24.67 3.47.67h.01c4.97 0 9-3.58 9-8s-4.03-8-9-8z"></path></svg>
      </div>
    </div>
  </header>

  <main>
    <!-- FEED -->
    <section id="tab-feed" class="tab active">
      <div class="stories-wrap">
        <div class="stories" id="stories"></div>
      </div>
      <div class="feed" id="feed"></div>
    </section>

    <!-- POST -->
    <section id="tab-post" class="tab">
      <div class="post-composer">
        <textarea id="postText" placeholder="Wetin dey sup?" maxlength="500"></textarea>
      </div>
      <button id="postBtn" class="post-btn" disabled="">Post</button>
    </section>

    <!-- CHAT -->
    <section id="tab-chat" class="tab">
      <div class="placeholder">Chats coming soon</div>
    </section>

    <!-- ME -->
    <section id="tab-me" class="tab">
      <div id="login-container">
        <div class="me-card">
          <h3 id="login-title">Login with Telegram</h3>
          <p>Connect your Telegram to start posting</p>
          <div id="telegram-login"></div>
        </div>
      </div>
      <div id="profile-container" style="display:none"></div>
    </section>
  </main>

  <nav class="bottom-nav">
    <div class="nav-inner">
      <button class="nav-btn active" data-tab="feed">
        <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2.1L1 12h3v9h6v-6h4v6h6v-9h3L12 2.1z"></path></svg>
        <span>Feed</span>
      </button>
      <button class="nav-btn" data-tab="post">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="3"></rect><path d="M12 8v8M8 12h8"></path></svg>
        <span>Post</span>
      </button>
      <button class="nav-btn" data-tab="chat">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7a8.38 8.38 0 0 1-.9-3.8 8.5 0 0 1 4.7-7.6 8.38 8.38 0 0 1 3.8-.9h.5a8.48 8.48 0 0 1 8 8v.5z"></path></svg>
        <span>Chat</span>
      </button>
      <button class="nav-btn" data-tab="me">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="8" r="4"></circle><path d="M4 20c0-4 3.5-6.5 8-6.5s8 2.5 8 6.5"></path></svg>
        <span>Me</span>
      </button>
    </div>
  </nav>
</div>

<div id="storyViewer" class="story-viewer"></div>

<script>
let currentUser = null;
let posts = [];
let stories = [];
let currentTab = 'feed';
let storyTimeout = null;

function $(s){ return document.querySelector(s) }
function escapeHtml(t){ const d=document.createElement('div'); d.textContent=t; return d.innerHTML }

function getInitials(name){ return name.split(' ').map(n=>n[0]).join('').toUpperCase().slice(0,2) }

function timeAgo(ts){
  const s = Math.floor((Date.now()-ts)/1000);
  if(s<60) return `${s}s`;
  const m = Math.floor(s/60); if(m<60) return `${m}m`;
  const h = Math.floor(m/60); if(h<24) return `${h}h`;
  const d = Math.floor(h/24); return `${d}d`;
}

function savePosts(){ localStorage.setItem('wachat_posts', JSON.stringify(posts)) }
function saveStories(){ localStorage.setItem('wachat_stories', JSON.stringify(stories)) }

function init(){
  try{
    currentUser = JSON.parse(localStorage.getItem('wachat_user')||'null');
    posts = JSON.parse(localStorage.getItem('wachat_posts')||'[]');
    stories = JSON.parse(localStorage.getItem('wachat_stories')||'[]');
  }catch(e){ posts=[]; stories=[] }
  stories = stories.filter(s=>Date.now()-s.time < 86400000);
  saveStories();
  renderFeed();
  bindNav();
  bindPost();
  window.addEventListener('wachat-login', e=>{ currentUser=e.detail; renderProfile(); switchTab('feed') });
  renderProfile();
}
function bindNav(){
  document.querySelectorAll('.nav-btn').forEach(btn=>{
    btn.addEventListener('click', ()=> switchTab(btn.dataset.tab));
  });
}
function switchTab(tab){
  if(tab==='post' && !currentUser){
    alert('Login first');
    tab='me';
  }
  document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
  $('#tab-'+tab).classList.add('active');
  document.querySelectorAll('.nav-btn').forEach(b=>b.classList.toggle('active', b.dataset.tab===tab));
  currentTab=tab;
  if(tab==='feed'){ renderStories(); renderFeed(); }
  if(tab==='post'){ setTimeout(()=>$('#postText').focus(),50) }
  if(tab==='me'){ renderProfile() }
  window.scrollTo(0,0);
}
function bindPost(){
  const ta=$('#postText'), btn=$('#postBtn');
  ta.addEventListener('input', ()=>{ btn.disabled = !ta.value.trim() });
  btn.addEventListener('click', handlePost);
}
function handlePost(){
  if(!currentUser){ alert('Login first'); switchTab('me'); return }
  const text=$('#postText').value.trim();
  if(!text) return;
  const post={ id:Date.now().toString(), userId:currentUser.id, name:currentUser.name, username:currentUser.username, avatar:currentUser.photo_url||'', text, time:Date.now(), likes:[], comments:[] };
  posts.unshift(post);
  savePosts();
  $('#postText').value='';
  $('#postBtn').disabled=true;
  switchTab('feed');
  renderFeed();
}
function renderStories(){
  const wrap=$('#stories'); wrap.innerHTML='';
  // Add your story
  const add=document.createElement('div'); add.className='story-item'; add.onclick=handleStory;
  add.innerHTML=`<div class="story-ring add"><div class="story-avatar add-inner"><svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="#1877f2" stroke-width="2.5"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg></div></div><div class="story-name">Your Story</div>`;
  wrap.appendChild(add);
  const valid=stories.filter(s=>Date.now()-s.time<86400000);
  const latest={}; valid.forEach(s=>{ if(!latest[s.userId]||s.time>latest[s.userId].time) latest[s.userId]=s });
  Object.values(latest).sort((a,b)=>b.time-a.time).forEach(s=>{
    const el=document.createElement('div'); el.className='story-item'; el.onclick=()=>openStory(s);
    const avatar = s.avatar ? `<img src="${s.avatar}" alt="">` : getInitials(s.name);
    el.innerHTML=`<div class="story-ring active"><div class="story-avatar">${avatar}</div></div><div class="story-name">${escapeHtml(s.name.split(' ')[0])}</div>`;
    wrap.appendChild(el);
  });
}
function handleStory(){
  if(!currentUser){ alert('Login first'); switchTab('me'); return }
  const text=prompt('Add your story (max 100 chars):');
  if(!text) return;
  if(text.length>100){ alert('Max 100 characters'); return }
  const s={ id:Date.now().toString(), userId:currentUser.id, name:currentUser.name, avatar:currentUser.photo_url||'', text:text.trim(), time:Date.now() };
  stories.unshift(s); saveStories(); renderStories();
}
function openStory(s){
  const v=$('#storyViewer'); v.classList.add('active');
  const avatar=s.avatar?`<img src="${s.avatar}">`:getInitials(s.name);
  v.innerHTML=`<div class="story-progress"><div class="story-progress-bar"></div></div>
    <div class="story-header">
      <div class="sv-avatar">${avatar}</div>
      <div class="sv-meta"><div class="sv-name">${escapeHtml(s.name)}</div><div class="sv-time">${timeAgo(s.time)}</div></div>
      <button class="sv-close">×</button>
    </div>
    <div class="story-content"><div class="story-text">${escapeHtml(s.text)}</div></div>`;
  clearTimeout(storyTimeout);
  storyTimeout=setTimeout(closeStory,5000);
  v.onclick=(e)=>{ if(e.target===v || e.target.closest('.sv-close') || e.target.closest('.story-content')) closeStory() };
}
function closeStory(){ $('#storyViewer').classList.remove('active'); clearTimeout(storyTimeout) }

function renderFeed(){
  const feed=$('#feed');
  if(posts.length===0){ feed.innerHTML=`<div class="empty-state">No posts yet. Be the first!</div>`; return }
  feed.innerHTML='';
  posts.forEach(p=>{
    const liked=currentUser && p.likes.includes(currentUser.id);
    const avatar = p.avatar ? `<img src="${p.avatar}">` : getInitials(p.name);
    const myAvatar = currentUser ? (currentUser.photo_url?`<img src="${currentUser.photo_url}">`:getInitials(currentUser.name)) : 'G';
    const card=document.createElement('article'); card.className='post-card';
    card.innerHTML=`
      <div class="post-head">
        <div class="post-avatar">${avatar}</div>
        <div class="post-meta">
          <div class="post-name">${escapeHtml(p.name)}</div>
          <div class="post-time">${timeAgo(p.time)} • @${escapeHtml(p.username)}</div>
        </div>
      </div>
      <div class="post-text">${escapeHtml(p.text)}</div>
      <div class="post-divider"></div>
      <div class="post-actions">
        <button class="action-btn ${liked?'liked':''}" onclick="handleLike('${p.id}')">
          <svg viewBox="0 0 24 24" fill="${liked?'currentColor':'none'}" stroke="currentColor" stroke-width="2"><path d="M20.84 4.61a5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 0 0 0 0-7.78z"/></svg>
          Like
        </button>
        <span class="comments-count">${p.comments.length} comments</span>
      </div>
      <div class="comment-bar">
        <div class="comment-avatar">${myAvatar}</div>
        <input class="comment-input" id="cmt-${p.id}" placeholder="Write a comment..." onkeydown="if(event.key==='Enter')handleComment('${p.id}')">
        <button class="comment-send" onclick="handleComment('${p.id}')">Send</button>
      </div>`;
    feed.appendChild(card);
  });
}
function handleLike(id){
  if(!currentUser){ alert('Login first'); switchTab('me'); return }
  const p=posts.find(x=>x.id===id); if(!p) return;
  const i=p.likes.indexOf(currentUser.id);
  if(i>-1) p.likes.splice(i,1); else p.likes.push(currentUser.id);
  savePosts(); renderFeed();
}
function handleComment(id){
  if(!currentUser){ alert('Login first'); switchTab('me'); return }
  const inp=document.getElementById('cmt-'+id); const text=inp.value.trim(); if(!text) return;
  const p=posts.find(x=>x.id===id); p.comments.push({id:Date.now().toString(), userId:currentUser.id, name:currentUser.name, text, time:Date.now()});
  savePosts(); inp.value=''; renderFeed();
}
function renderProfile(){
  const loginC=$('#login-container'), profileC=$('#profile-container');
  if(currentUser){
    loginC.style.display='none'; profileC.style.display='block';
    const avatar = currentUser.photo_url?`<img src="${currentUser.photo_url}">`:getInitials(currentUser.name);
    profileC.innerHTML=`<div class="me-card">
      <div class="profile-avatar">${avatar}</div>
      <div class="profile-name">${escapeHtml(currentUser.name)}</div>
      <div class="profile-username">@${escapeHtml(currentUser.username)}</div>
      <div class="green-badge">✓ Logged in with Telegram</div>
      <button class="logout-btn" onclick="logout()">Logout</button>
    </div>`;
  }else{
    loginC.style.display='block'; profileC.style.display='none';
    const box=$('#telegram-login');
    if(!box.querySelector('script')){
      const s=document.createElement('script');
      s.async=true; s.src="https://telegram.org/js/telegram-widget.js?22";
      s.setAttribute('data-telegram-login','Wchart312_bot');
      s.setAttribute('data-size','large');
      s.setAttribute('data-onauth','onTelegramAuth(user)');
      s.setAttribute('data-request-access','write');
      box.innerHTML=''; box.appendChild(s);
    }
  }
}
function logout(){ localStorage.removeItem('wachat_user'); currentUser=null; renderProfile(); switchTab('feed') }

// dev triple-tap login for testing
let taps=0; document.addEventListener('DOMContentLoaded', ()=>{
  $('#login-title')?.addEventListener('click', ()=>{ taps++; if(taps>=3){ onTelegramAuth({id:Date.now(), first_name:'Test', last_name:'User', username:'testuser', photo_url:''}) } });
});

document.addEventListener('DOMContentLoaded', init);
</script>
<script>(function(){document.addEventListener("click",function(e){var a=e.target.closest("[data-product-id]");if(!a)return;e.preventDefault();var pid=a.getAttribute("data-product-id");if(pid)parent.postMessage({type:"ecto-artifact-link-click",productId:pid},"*")})})();</script>

</body></html>
