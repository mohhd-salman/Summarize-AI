<script>
  import { currentStep } from "../stores/appStore";

  function resetApp() {
    currentStep.set(1);
  }

  function handleKeyDown(event) {
    if (event.key === "Enter" || event.key === " ") {
      resetApp();
    }
  }
</script>

<header>
  <div class="container header-content">
    <a
            href="/"
            class="logo"
            on:keydown={handleKeyDown}
            tabindex="0"
            role="button"
            aria-label="Go to Home"
    >
      <h1>Summarize AI</h1>
    </a>

    <nav>
      <ul>
        <li>
          <a
            href="https://github.com/mohhd-salman/Summarize-AI"
            target="_blank"
            rel="noopener noreferrer">GitHub</a
          >
        </li>
      </ul>
    </nav>
  </div>
</header>

<style>
  a.logo {
  text-decoration: none;
  color: inherit;
}
  header {
    background-color: white;
    box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
    padding: 1rem 0;
  }

  .header-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .logo {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    cursor: pointer;
  }


  .logo h1 {
    font-size: 1.5rem;
    margin: 0;
    color: var(--primary-color);
  }

  nav ul {
    list-style: none;
    display: flex;
    gap: 1.5rem;
  }

  nav a {
    color: var(--text-color);
    font-weight: 500;
    transition: color 0.2s;
  }

  nav a:hover {
    color: var(--primary-color);
    text-decoration: none;
  }
</style>
