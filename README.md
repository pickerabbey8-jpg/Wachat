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
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
<title>Wachat</title>
<style>
*{box-sizing:border-box;margin:0;padding:0;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial}
body{background:#0b141a;color:#e9edef;height:100vh;display:flex;flex-direction:column}
header{background:#202c33;padding:14px;text-align:center;font-weight:700;font-size:18px;color:#00a884}
main{flex:1;overflow-y:auto;padding:12px;padding-bottom:80px}
.tab{display:none}
.tab.active{display:block}
.card{background:#1f2c34;border-radius:12px;padding:14px;margin-bottom:12px}
.btn{background:#00a884;color:#000;border:none;padding:12px 18px;border-radius:8px;font-weight:600;width:100%;margin-top:10px}
.input,textarea{width:100%;background:#2a3942;border:none;color:#e9edef;padding:12px;border-radius:8px;margin-top:8px}
nav{position:fixed;bottom:0;left:0;right:0;background:#202c33;display:flex;border-top:1px solid #2a3942}
nav button{flex:1;background:none;border:none;color:#8696a0;padding:12px;font-size:13px}
nav button.active{color:#00a884}
.avatar{width:60px;height:60px;border-radius:50%;background:#00a884;display:flex;align-items:center;justify-content:center;font-size:24px;margin:0 auto 10px}
.post-user{font-weight:600;color:#00a884;margin-bottom:4px}
.post-time{font-size:11px;color:#8696a0;margin-bottom:8px}
.empty{color:#8696a0;text-align:center;margin-top:40px}
</style>
</head>
<body>
<header>Wachat</header>
<main>
  <div id="me" class="tab active">
    <div class="card" style="text-align:center">
      <div id="profile">
        <div class="avatar" id="av">?</div>
        <div id="name" style="font-weight:600">Not logged in</div>
        <div id="uname" style="color:#8696a0;font-size:13px">@guest</div>
      </div>
      <div id="loginArea"><iframe id="telegram-login-Wchart312_bot" src="https://oauth.telegram.org/embed/Wchart312_bot?origin=https%3A%2F%2Fembed.fbsbx.com&amp;return_to=https%3A%2F%2Fembed.fbsbx.com%2Fplayables%2Fview%2F939332019157089%2F%3Fext%3D1789911858%26hash%3DQ92gDAHOj0MYEPCIZwSEWvZs1s_v&amp;size=large&amp;request_access=write" width="238" height="40" frameborder="0" scrolling="no" style="overflow: hidden; color-scheme: light dark; border: none;"></iframe><script src="https://telegram.org/js/telegram-widget.js?22" data-telegram-login="Wchart312_bot" data-size="large" data-onauth="onTelegramAuth(user)" data-request-access="write"></script></div>
      <button class="btn" id="logoutBtn" style="display:none;background:#2a3942;color:#e9edef" onclick="logout()">Logout</button>
    </div>
  </div>

  <div id="feed" class="tab">
    <div id="feedList"></div>
    <div id="feedEmpty" class="empty" style="display: none;">No posts yet. Be the first!</div>
  </div>

  <div id="post" class="tab">
    <div class="card">
      <h3>New Post</h3>
      <textarea id="postText" class="input" rows="4" placeholder="Wetin dey sup?"></textarea>
      <button class="btn" onclick="createPost()">Post</button>
    </div>
  </div>

  <div id="chat" class="tab">
    <div class="card">
      <h3>Chats</h3>
      <div class="empty">Chats coming soon – DM go work with real backend later</div>
    </div>
  </div>
</main>

<nav>
  <button class="active" onclick="showTab('me',this)">Me</button>
  <button onclick="showTab('feed',this)">Feed</button>
  <button onclick="showTab('post',this)">Post</button>
  <button onclick="showTab('chat',this)">Chat</button>
</nav>

<script async="" src="https://telegram.org/js/telegram-widget.js?22" onload="initLogin()"></script>
<script>
const BOT = 'Wchart312_bot';
let user = JSON.parse(localStorage.getItem('wachat_user')||'null');
let posts = JSON.parse(localStorage.getItem('wachat_posts')||'[]'); // NO DEMO POSTS

function initLogin(){
  const area = document.getElementById('loginArea');
  if(user){ renderProfile(); return; }
  area.innerHTML = '';
  const s = document.createElement('script');
  s.src = 'https://telegram.org/js/telegram-widget.js?22';
  s.setAttribute('data-telegram-login', BOT);
  s.setAttribute('data-size', 'large');
  s.setAttribute('data-onauth', 'onTelegramAuth(user)');
  s.setAttribute('data-request-access', 'write');
  area.appendChild(s);
}

function onTelegramAuth(u){
  user = u;
  localStorage.setItem('wachat_user', JSON.stringify(u));
  renderProfile();
  document.getElementById('loginArea').innerHTML = '';
  document.getElementById('logoutBtn').style.display = 'block';
}

function renderProfile(){
  if(!user) return;
  document.getElementById('av').innerHTML = user.photo_url ? `<img src="${user.photo_url}" style="width:100%;height:100%;border-radius:50%">` : user.first_name[0];
  document.getElementById('name').textContent = user.first_name + (user.last_name?' '+user.last_name:'');
  document.getElementById('uname').textContent = '@' + (user.username||user.id);
  document.getElementById('logoutBtn').style.display = 'block';
}

function logout(){
  localStorage.removeItem('wachat_user');
  user = null;
  location.reload();
}

function showTab(id,btn){
  document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  document.querySelectorAll('nav button').forEach(b=>b.classList.remove('active'));
  btn.classList.add('active');
  if(id==='feed') renderFeed();
}

function createPost(){
  if(!user){ alert('Login first for Me tab'); showTab('me',document.querySelector('nav button')); return; }
  const txt = document.getElementById('postText').value.trim();
  if(!txt) return;
  const p = {id:Date.now(), user:user.first_name, uname:user.username||user.id, text:txt, time:new Date().toLocaleString(), avatar:user.photo_url};
  posts.unshift(p);
  localStorage.setItem('wachat_posts', JSON.stringify(posts));
  document.getElementById('postText').value='';
  showTab('feed',document.querySelectorAll('nav button')[1]);
}

function renderFeed(){
  const list = document.getElementById('feedList');
  list.innerHTML = '';
  if(posts.length===0){ document.getElementById('feedEmpty').style.display='block'; return; }
  document.getElementById('feedEmpty').style.display='none';
  posts.forEach(p=>{
    const d = document.createElement('div');
    d.className='card';
    d.innerHTML = `<div style="display:flex;gap:10px"><div style="width:36px;height:36px;border-radius:50%;background:#00a884;display:flex;align-items:center;justify-content:center;overflow:hidden">${p.avatar?`<img src="${p.avatar}" style="width:100%">`:p.user[0]}</div><div style="flex:1"><div class="post-user">${p.user}</div><div class="post-time">@${p.uname} • ${p.time}</div><div>${p.text}</div></div></div>`;
    list.appendChild(d);
  });
}

if(user) renderProfile();
renderFeed();
</script>


</body></html>
