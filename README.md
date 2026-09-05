<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>対話</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Shippori+Mincho:wght@500;700&family=Zen+Kaku+Gothic+New:wght@400;500;700&display=swap" rel="stylesheet">
<style>
  :root {
    --ink: #14213D;
    --ink-deep: #0C1526;
    --paper: #F2F0E8;
    --gold: #C99B3F;
    --line: rgba(20, 33, 61, 0.14);
    --muted: #6B7280;
  }

  * { box-sizing: border-box; }

  html, body {
    height: 100%;
    margin: 0;
  }

  body {
    background: var(--ink-deep);
    color: var(--paper);
    font-family: 'Zen Kaku Gothic New', sans-serif;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  header {
    padding: 28px 32px 20px;
    border-bottom: 1px solid rgba(242, 240, 232, 0.1);
    flex-shrink: 0;
  }

  header h1 {
    font-family: 'Shippori Mincho', serif;
    font-weight: 700;
    font-size: 22px;
    letter-spacing: 0.04em;
    margin: 0;
    color: var(--paper);
  }

  header p {
    margin: 6px 0 0;
    font-size: 13px;
    color: rgba(242, 240, 232, 0.5);
  }

  #scroll {
    flex: 1;
    overflow-y: auto;
    padding: 24px 0 8px;
  }

  .row {
    max-width: 680px;
    margin: 0 auto;
    padding: 0 32px;
    display: flex;
    margin-bottom: 22px;
  }

  .row.user { justify-content: flex-end; }
  .row.assistant { justify-content: flex-start; }

  .bubble {
    max-width: 78%;
    padding: 14px 18px;
    line-height: 1.7;
    font-size: 15px;
    white-space: pre-wrap;
    word-break: break-word;
  }

  .row.user .bubble {
    background: var(--gold);
    color: var(--ink-deep);
    border-radius: 16px 16px 4px 16px;
    font-weight: 500;
  }

  .row.assistant .bubble {
    background: rgba(242, 240, 232, 0.06);
    border: 1px solid rgba(242, 240, 232, 0.12);
    color: var(--paper);
    border-radius: 16px 16px 16px 4px;
  }

  .row.assistant .label {
    font-family: 'Shippori Mincho', serif;
    font-size: 11px;
    color: var(--gold);
    margin-bottom: 6px;
    letter-spacing: 0.08em;
  }

  .typing {
    display: inline-flex;
    gap: 4px;
    align-items: center;
    padding: 4px 0;
  }
  .typing span {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: var(--gold);
    opacity: 0.5;
    animation: blink 1.2s infinite ease-in-out;
  }
  .typing span:nth-child(2) { animation-delay: 0.2s; }
  .typing span:nth-child(3) { animation-delay: 0.4s; }
  @keyframes blink {
    0%, 80%, 100% { opacity: 0.25; transform: translateY(0); }
    40% { opacity: 1; transform: translateY(-2px); }
  }

  .empty {
    max-width: 680px;
    margin: 60px auto;
    padding: 0 32px;
    text-align: center;
    color: rgba(242, 240, 232, 0.35);
  }
  .empty .mark {
    font-family: 'Shippori Mincho', serif;
    font-size: 34px;
    color: rgba(201, 155, 63, 0.7);
    margin-bottom: 10px;
  }

  form {
    flex-shrink: 0;
    max-width: 680px;
    width: 100%;
    margin: 0 auto;
    padding: 16px 32px 28px;
    display: flex;
    gap: 10px;
    align-items: flex-end;
  }

  textarea {
    flex: 1;
    resize: none;
    background: rgba(242, 240, 232, 0.06);
    border: 1px solid rgba(242, 240, 232, 0.16);
    border-radius: 12px;
    color: var(--paper);
    font-family: inherit;
    font-size: 15px;
    padding: 12px 14px;
    line-height: 1.5;
    max-height: 140px;
    outline: none;
    transition: border-color 0.15s ease;
  }
  textarea:focus { border-color: var(--gold); }
  textarea::placeholder { color: rgba(242, 240, 232, 0.35); }

  button {
    background: var(--gold);
    color: var(--ink-deep);
    border: none;
    border-radius: 12px;
    padding: 12px 20px;
    font-family: 'Zen Kaku Gothic New', sans-serif;
    font-weight: 700;
    font-size: 14px;
    cursor: pointer;
    transition: filter 0.15s ease, opacity 0.15s ease;
  }
  button:hover { filter: brightness(1.08); }
  button:disabled { opacity: 0.4; cursor: not-allowed; }

  #scroll::-webkit-scrollbar { width: 8px; }
  #scroll::-webkit-scrollbar-thumb { background: rgba(242, 240, 232, 0.12); border-radius: 4px; }

  @media (max-width: 520px) {
    header { padding: 20px 18px 16px; }
    .row { padding: 0 18px; }
    form { padding: 12px 18px 20px; }
    .bubble { max-width: 88%; }
  }
</style>
</head>
<body>

<header>
  <h1>対話</h1>
  <p>思ったことをそのまま書いてください</p>
</header>

<div id="scroll">
  <div class="empty" id="emptyState">
    <div class="mark">—</div>
    <div>まだ会話がありません。下から話しかけてみてください。</div>
  </div>
</div>

<form id="form">
  <textarea id="input" placeholder="メッセージを入力…" rows="1"></textarea>
  <button type="submit" id="sendBtn">送る</button>
</form>

<script>
  const scrollEl = document.getElementById('scroll');
  const form = document.getElementById('form');
  const input = document.getElementById('input');
  const sendBtn = document.getElementById('sendBtn');
  const emptyState = document.getElementById('emptyState');

  let history = [];
  let busy = false;

  input.addEventListener('input', () => {
    input.style.height = 'auto';
    input.style.height = Math.min(input.scrollHeight, 140) + 'px';
  });

  input.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      form.requestSubmit();
    }
  });

  function addRow(role, text) {
    if (emptyState) { emptyState.remove(); }
    const row = document.createElement('div');
    row.className = 'row ' + role;

    const bubble = document.createElement('div');
    bubble.className = 'bubble';

    if (role === 'assistant') {
      const label = document.createElement('div');
      label.className = 'label';
      label.textContent = 'AI';
      bubble.appendChild(label);
      const textNode = document.createElement('span');
      textNode.textContent = text;
      bubble.appendChild(textNode);
    } else {
      bubble.textContent = text;
    }

    row.appendChild(bubble);
    scrollEl.appendChild(row);
    scrollEl.scrollTop = scrollEl.scrollHeight;
    return bubble;
  }

  function addTypingRow() {
    if (emptyState) { emptyState.remove(); }
    const row = document.createElement('div');
    row.className = 'row assistant';
    row.id = 'typingRow';

    const bubble = document.createElement('div');
    bubble.className = 'bubble';
    const label = document.createElement('div');
    label.className = 'label';
    label.textContent = 'AI';
    bubble.appendChild(label);

    const typing = document.createElement('div');
    typing.className = 'typing';
    typing.innerHTML = '<span></span><span></span><span></span>';
    bubble.appendChild(typing);

    row.appendChild(bubble);
    scrollEl.appendChild(row);
    scrollEl.scrollTop = scrollEl.scrollHeight;
  }

  function removeTypingRow() {
    const row = document.getElementById('typingRow');
    if (row) row.remove();
  }

  async function sendMessage(text) {
    history.push({ role: 'user', content: text });
    addTypingRow();
    busy = true;
    sendBtn.disabled = true;

    const MAX_ATTEMPTS = 3;
    let lastErr = null;

    for (let attempt = 1; attempt <= MAX_ATTEMPTS; attempt++) {
      try {
        const response = await fetch('https://api.anthropic.com/v1/messages', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            model: 'claude-sonnet-4-6',
            max_tokens: 1000,
            messages: history
          })
        });

        let data;
        try {
          data = await response.json();
        } catch (parseErr) {
          throw new Error('サーバーからの応答を解析できませんでした。');
        }

        if (!response.ok) {
          // 過負荷やレート制限は少し待って再試行
          if ((response.status === 429 || response.status === 529 || response.status >= 500) && attempt < MAX_ATTEMPTS) {
            lastErr = new Error(data?.error?.message || `HTTP ${response.status}`);
            await new Promise(r => setTimeout(r, attempt * 1000));
            continue;
          }
          const msg = data?.error?.message || `サーバーエラー(${response.status})`;
          throw new Error(msg);
        }

        removeTypingRow();

        let replyText = '';
        if (data && data.content) {
          replyText = data.content
            .filter(block => block.type === 'text')
            .map(block => block.text)
            .join('\n');
        }
        if (!replyText) {
          replyText = 'すみません、応答が空でした。もう一度お試しください。';
        }

        history.push({ role: 'assistant', content: replyText });
        addRow('assistant', replyText);
        busy = false;
        sendBtn.disabled = false;
        input.focus();
        return;

      } catch (err) {
        lastErr = err;
        // ネットワーク自体が落ちた場合のみ待って再試行、それ以外は即エラー表示
        if (attempt < MAX_ATTEMPTS && err.message && !err.message.includes('サーバーエラー')) {
          await new Promise(r => setTimeout(r, attempt * 800));
          continue;
        }
        break;
      }
    }

    removeTypingRow();
    // 会話履歴から失敗した送信分を取り除き、再送できるようにする
    history.pop();
    const detail = lastErr && lastErr.message ? lastErr.message : '不明なエラー';
    addRow('assistant', `通信エラーが発生しました(${detail})。もう一度お試しください。`);
    console.error(lastErr);
    busy = false;
    sendBtn.disabled = false;
    input.focus();
  }

  form.addEventListener('submit', (e) => {
    e.preventDefault();
    const text = input.value.trim();
    if (!text || busy) return;

    addRow('user', text);
    input.value = '';
    input.style.height = 'auto';
    sendMessage(text);
  });
</script>

</body>
</html>
