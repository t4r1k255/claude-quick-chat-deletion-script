// ==UserScript==
// @name         Claude – Hızlı Chat Sil
// @namespace    https://claude.ai/
// @version      5.0
// @author       t4r1k255 (https://github.com/t4r1k255)
// @homepageURL  https://github.com/t4r1k255/claude-hizli-sil
// @description  Tek tıkla sil + ghost entry'leri kalıcı olarak temizle
// @match        https://claude.ai/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function () {
    'use strict';

    const _fetch = window.fetch.bind(window);
    let learnedBase = null;
    let capturedHeaders = {};
    let allOrgs = [];

    // ── localStorage tabanlı silme kaydı ──────────────────────────
    // Sunucuda silinen ama React'ın tekrar render ettiği "ghost" entry'leri
    // önlemek için silinen ID'leri localStorage'da tutuyoruz.
    const LS_KEY = 'claude_qs_deleted';

    function getDeleted() {
        try { return new Set(JSON.parse(localStorage.getItem(LS_KEY) || '[]')); }
        catch { return new Set(); }
    }

    function markDeleted(id) {
        const arr = [...getDeleted(), id];
        // Max 500 kayıt (~18 KB) — daha eskiler otomatik çıkar
        localStorage.setItem(LS_KEY, JSON.stringify(arr.slice(-500)));
    }

    // ── Fetch proxy (auth header toplama + endpoint öğrenme) ───────
    window.fetch = function (input, init = {}) {
        const url = typeof input === 'string' ? input : (input?.url ?? '');
        const method = (init?.method ?? 'GET').toUpperCase();

        if (/\/api\//i.test(url) && init?.headers) {
            try {
                const h = init.headers instanceof Headers
                    ? Object.fromEntries(init.headers)
                    : { ...init.headers };
                for (const [k, v] of Object.entries(h))
                    if (/^(authorization|x-|anthropic-)/i.test(k))
                        capturedHeaders[k] = v;
            } catch {}
        }

        if (method === 'DELETE' && /chat_conversation/i.test(url)) {
            const m = url.match(/^(.*\/chat_conversations\/)[^/?#]*/);
            if (m) {
                learnedBase = m[1];
                console.log('[Hızlı Sil] ✓ Endpoint öğrenildi:', learnedBase);
            }
        }

        return _fetch(input, init);
    };

    // ── Init ───────────────────────────────────────────────────────
    function init() {
        (async () => {
            try {
                const r = await _fetch('/api/organizations', { credentials: 'include' });
                if (!r.ok) return;
                allOrgs = await r.json();
                console.log('[Hızlı Sil] ✓ Org sayısı:', allOrgs.length);
                if (allOrgs[0]?.uuid && !learnedBase)
                    learnedBase = `/api/organizations/${allOrgs[0].uuid}/chat_conversations/`;
            } catch (e) { console.warn('[Hızlı Sil] org hatası:', e); }
        })();

        function scan() {
            const deleted = getDeleted();

            document.querySelectorAll('a[href*="/chat/"]').forEach(a => {
                const m = a.href.match(/\/chat\/([^/?#]+)/);
                if (!m) return;
                const id = m[1];

                const li = a.closest('li') || a.parentElement;
                if (!li || li === document.body) return;

                // Ghost entry: sunucuda silinmiş ama React tekrar render etmiş
                if (deleted.has(id)) {
                    li.remove();
                    return;
                }

                addButton(li, id);
            });
        }

        new MutationObserver(scan).observe(document.body, { childList: true, subtree: true });
        scan();
    }

    document.readyState === 'loading'
        ? document.addEventListener('DOMContentLoaded', init)
        : init();

    // ── DELETE isteği ──────────────────────────────────────────────
    async function deleteConv(id) {
        const bases = learnedBase
            ? [learnedBase]
            : allOrgs.map(o => `/api/organizations/${o.uuid}/chat_conversations/`);

        for (const base of bases) {
            const url = base + id;
            console.log('[Hızlı Sil] → DELETE', url);
            const r = await _fetch(url, {
                method: 'DELETE',
                credentials: 'include',
                headers: capturedHeaders,
            });
            const body = await r.text().catch(() => '');
            console.log('[Hızlı Sil] ←', r.status, body.slice(0, 120));
            if (r.ok) return;
            if (r.status !== 404) throw new Error(`HTTP ${r.status}: ${body.slice(0, 150)}`);
        }
        throw new Error('404 – tüm org\'lar denendi');
    }

    // ── Buton ekle ─────────────────────────────────────────────────
    function addButton(container, convId) {
        if (container._qsDel) return;
        container._qsDel = true;

        if (getComputedStyle(container).position === 'static')
            container.style.position = 'relative';

        const btn = document.createElement('button');
        btn.textContent = '✕';
        btn.title = 'Sil (onaysız)';
        Object.assign(btn.style, {
            position: 'absolute', left: '4px', top: '50%',
            transform: 'translateY(-50%)', zIndex: '50',
            width: '20px', height: '20px', padding: '0',
            background: 'transparent', border: 'none',
            borderRadius: '4px', color: '#f87171',
            fontSize: '11px', fontWeight: '700',
            cursor: 'pointer', opacity: '0',
            transition: 'opacity .15s, background .15s',
        });

        container.addEventListener('mouseenter', () => btn.style.opacity = '1');
        container.addEventListener('mouseleave', () => btn.style.opacity = '0');
        btn.addEventListener('mouseenter', () => btn.style.background = 'rgba(239,68,68,.15)');
        btn.addEventListener('mouseleave', () => btn.style.background = 'transparent');

        btn.addEventListener('click', async e => {
            e.preventDefault();
            e.stopPropagation();
            btn.textContent = '…';
            btn.style.pointerEvents = 'none';
            try {
                await deleteConv(convId);
                markDeleted(convId);                              // ← localStorage'a yaz
                (container.closest('li') || container).remove(); // ← anlık DOM temizliği
                // O sohbeti açıkken sildiysen anasayfaya dön
                if (location.pathname.includes(convId))
                    location.replace('/');
            } catch (err) {
                console.error('[Hızlı Sil] HATA:', err.message);
                btn.textContent = '!';
                btn.title = err.message;
                setTimeout(() => {
                    btn.textContent = '✕';
                    btn.title = 'Sil (onaysız)';
                    btn.style.pointerEvents = '';
                }, 3000);
            }
        });

        container.appendChild(btn);
    }
})();
