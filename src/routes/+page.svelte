<script lang="ts">
  import Button from "$lib/components/Button.svelte";
  import { onMount } from "svelte";
  import { History, Delete } from "lucide-svelte";

  let display = $state("");
  let result = $state("");
  let history = $state<string[]>([]);
  let calculated = $state(false);
  let showHistory = $state(false);

  const buttons = [
    { value: "C" },
    { value: "(" },
    { value: ")" },
    { value: "delete", icon: Delete },

    { value: "%" },
    { value: "÷" },
    { value: "×" },
    { value: "-" },

    { value: "7" },
    { value: "8" },
    { value: "9" },
    { value: "+" },

    { value: "4" },
    { value: "5" },
    { value: "6" },
    { value: "." },

    { value: "1" },
    { value: "2" },
    { value: "3" },
    { value: "=" },

    { value: "0" },
  ];

  const inputKeys = "0123456789().%";
  const operatorKeys = "+-*/";

  onMount(() => {
    const buttonElements = () =>
      Array.from(
        document.querySelectorAll(".buttons button"),
      ) as HTMLButtonElement[];

    function moveFocus(direction: "up" | "down" | "left" | "right") {
      const buttons = buttonElements();
      const currentIndex = buttons.indexOf(
        document.activeElement as HTMLButtonElement,
      );

      if (currentIndex === -1) return;
      const movement = {
        right: currentIndex + 1,
        left: currentIndex - 1,
        down: currentIndex + 4,
        up: currentIndex - 4,
      };

      let nextIndex = movement[direction];

      if (direction === "right") {
        nextIndex %= buttons.length;
      }

      if (direction === "left") {
        nextIndex = (nextIndex + buttons.length) % buttons.length;
      }

      nextIndex = Math.max(0, Math.min(nextIndex, buttons.length - 1));
      buttons[nextIndex]?.focus();
    }

    function handleKey(event: KeyboardEvent) {
      const key = event.key;

      switch (key) {
        case "ArrowRight":
          event.preventDefault();
          moveFocus("right");
          return;

        case "ArrowLeft":
          event.preventDefault();
          moveFocus("left");
          return;

        case "ArrowDown":
          event.preventDefault();
          moveFocus("down");
          return;

        case "ArrowUp":
          event.preventDefault();
          moveFocus("up");
          return;
      }

      if (
        key === "Enter" &&
        document.activeElement instanceof HTMLButtonElement
      ) {
        return;
      }

      if (showHistory && key === "Escape") {
        toggleHistory();
        return;
      }

      if (inputKeys.includes(key)) {
        press(key);
        return;
      }

      if (operatorKeys.includes(key)) {
        press(key);
        return;
      }

      switch (key) {
        case "Enter":
          calculate();
          break;

        case "Backspace":
          removeLast();
          break;

        case "Escape":
          clearDisplay();
          break;
      }
    }

    window.addEventListener("keydown", handleKey);
    return () => {
      window.removeEventListener("keydown", handleKey);
    };
  });

  function handleButton(value: string) {
    switch (value) {
      case "C":
        clearDisplay();
        return;

      case "=":
        calculate();
        return;

      case "delete":
        removeLast();
        return;

      case "÷":
        press("/");
        return;

      case "×":
        press("*");
        return;

      default:
        press(value);
    }
  }

  function press(value: string) {
    if (calculated) {
      display = operatorKeys.includes(value) ? result + value : value;
      if (!operatorKeys.includes(value)) {
        result = "";
      }
      calculated = false;
      return;
    }
    display += value;
  }

  function handleHistoryKey(e: KeyboardEvent) {
    if (e.key === "Escape" || e.key === "Enter") {
      toggleHistory();
    }
  }

  function removeLast() {
    display = display.slice(0, -1);
  }

  function clearDisplay() {
    display = "";
    result = "";
    calculated = false;
  }

  function toggleHistory() {
    showHistory = !showHistory;
  }

  function calculate() {
    if (!display.trim()) return;

    try {
      const expression = display
        .replace(/×/g, "*")
        .replace(/÷/g, "/")
        .replace(/%/g, "/100");
      if (/[+\-*/.]$/.test(expression)) {
        result = "Error";
        return;
      }

      const value = Function(`"use strict"; return (${expression})`)();
      if (
        value === undefined ||
        Number.isNaN(value) ||
        !Number.isFinite(value)
      ) {
        result = "Error";
        return;
      }
      result = value.toString();
      history = [`${display} = ${result}`, ...history];
      calculated = true;
    } catch {
      result = "Error";
    }
  }
</script>

<div class="calculator-layout">
  <div class="calculator">
    <div class="display-header mobile-only">
      <button
        class="history-btn"
        type="button"
        onclick={toggleHistory}
        aria-label="Open History"
      >
        <History size={22} />
      </button>
    </div>

    <div class="display-box">
      <p class="expression">
        {display || "0"}
      </p>

      <h1 class="result">
        {result || "0"}
      </h1>
    </div>

    <div class="buttons">
      {#each buttons as btn}
        <div
          class:equal-btn={btn.value === "="}
          class:zero-btn={btn.value === "0"}
        >
          <Button
            value={btn.value}
            icon={btn.icon}
            onClick={() => handleButton(btn.value)}
          />
        </div>
      {/each}
    </div>
  </div>

  <div class="desktop-history">
    <div class="history-header">
      <h3>History</h3>
    </div>

    {#if history.length}
      {#each history as item}
        <div class="history-item">
          {item}
        </div>
      {/each}
    {:else}
      <p class="empty-history">No History Yet</p>
    {/if}
  </div>

  {#if showHistory && window.innerWidth <= 768}
    <div
      class="history-backdrop"
      role="button"
      tabindex="0"
      onclick={toggleHistory}
      onkeydown={handleHistoryKey}
    >
      <div
        class="history-sheet"
        role="dialog"
        tabindex="-1"
        onmousedown={(e) => e.stopPropagation()}
      >
        <div class="drag-line"></div>

        <div class="history-header">
          <h3>History</h3>
        </div>

        {#if history.length}
          {#each history as item}
            <div class="history-item">
              {item}
            </div>
          {/each}
        {:else}
          <p class="empty-history">No History Yet</p>
        {/if}
      </div>
    </div>
  {/if}
</div>

<style>
  :global(body) {
    margin: 0;
    font-family: "Inter", "Segoe UI", sans-serif;
    background: linear-gradient(135deg, #eef2ff, #f8fafc);
    min-height: 100vh;
    overflow: hidden;
  }

  .calculator-layout {
    display: flex;
    height: 100vh;
    width: 100vw;
    padding: 12px;

    gap: 12px;
    box-sizing: border-box;
  }

  .calculator {
    flex: 1;
    max-width: none;
    height: calc(100vh - 24px);
    background: rgba(255, 255, 255, 0.92);
    backdrop-filter: blur(14px);
    border: 1px solid rgba(255, 255, 255, 0.5);
    border-radius: 20px;
    padding: 18px;
    box-shadow: 0 20px 50px rgba(15, 23, 42, 0.12);
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  .desktop-history {
    width: 320px;
    min-width: 320px;
    height: calc(100vh - 24px);
    background: rgba(255, 255, 255, 0.94);
    backdrop-filter: blur(14px);
    border-radius: 20px;
    padding: 18px;
    box-shadow: 0 20px 50px rgba(15, 23, 42, 0.1);
    display: flex;
    flex-direction: column;
    overflow-y: auto;
  }

  .display-header {
    display: flex;
    justify-content: flex-end;
    margin-bottom: 16px;
  }

  .mobile-only {
    display: none;
  }

  .history-btn {
    width: 46px;
    height: 46px;
    border: none;
    border-radius: 14px;
    background: #f1f5f9;
    color: #334155;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.25s ease;
  }

  .history-btn:hover {
    background: #e2e8f0;
    transform: translateY(-2px);
  }

  .display-box {
    background: linear-gradient(145deg, #ffffff, #f8fafc);
    border-radius: 18px;
    padding: 18px;
    min-height: 140px;
    margin-bottom: 16px;
    box-shadow: inset 0 2px 10px rgba(148, 163, 184, 0.15);
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    flex-shrink: 0;
  }

  .expression {
    font-size: 1.1rem;
    color: #64748b;
    text-align: right;
    margin: 0;
    word-break: break-word;
  }

  .result {
    font-size: 3rem;
    font-weight: 700;
    color: #0f172a;
    text-align: right;
    margin: 10px 0 0;
    word-break: break-word;
  }

  .buttons {
    flex: 1;
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(5, minmax(55px, 1fr));
    gap: 10px;
    min-height: 0;
  }

  .buttons > div {
    height: 100%;
  }

  .equal-btn :global(button) {
    background: linear-gradient(135deg, #2563eb, #1d4ed8);
    color: white;
    font-size: 1.3rem;
    font-weight: 700;
    box-shadow: 0 10px 20px rgba(37, 99, 235, 0.35);
  }

  .equal-btn :global(button):hover {
    background: linear-gradient(135deg, #1d4ed8, #1e40af);
  }

  .history-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }

  .history-header h3 {
    margin: 0;
    font-size: 1.2rem;
    color: #0f172a;
    font-weight: 700;
  }

  .history-item {
    background: #f8fafc;
    padding: 14px 16px;
    border-radius: 14px;
    margin-bottom: 12px;
    color: #334155;
    font-size: 0.95rem;
    box-shadow: inset 0 1px 4px rgba(148, 163, 184, 0.12);
    word-break: break-word;
  }

  .empty-history {
    text-align: center;
    color: #94a3b8;
    margin-top: 40px;
  }

  .history-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(15, 23, 42, 0.55);
    z-index: 999;
    display: flex;
    justify-content: center;
    align-items: flex-end;
  }

  .history-sheet {
    width: 100%;
    max-height: 75vh;
    background: white;
    border-radius: 24px 24px 0 0;
    padding: 18px;
    overflow-y: auto;
    animation: slideUp 0.3s ease;
  }

  .drag-line {
    width: 50px;
    height: 5px;
    background: #cbd5e1;
    border-radius: 999px;
    margin: 0 auto 18px;
  }

  @keyframes slideUp {
    from {
      transform: translateY(100%);
    }
    to {
      transform: translateY(0);
    }
  }

  @media (max-width: 768px) {
    :global(body) {
      overflow: auto;
    }

    .calculator-layout {
      height: auto;
      width: 100%;
      padding: 20px;
      flex-direction: column;
      align-items: center;
    }

    .desktop-history {
      display: none;
    }

    .mobile-only {
      display: flex;
    }

    .calculator {
      width: 100%;
      height: auto;
      padding: 18px;
      border-radius: 22px;
    }

    .display-box {
      min-height: 130px;
      padding: 20px;
    }

    .result {
      font-size: 2rem;
    }

    .buttons {
      grid-template-rows: auto;
    }
  }
</style>
