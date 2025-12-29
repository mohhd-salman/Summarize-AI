<footer class="custom-footer">
  <div class="container">
    <p>© {new Date().getFullYear()} Summarize AI App. All rights reserved.</p>
    <div class="footer-links">
      <a href="/">About</a>
      <a href="/">Privacy Policy</a>
      <a href="/">Contact</a>
    </div>
  </div>
</footer>

<style>
  footer.custom-footer {
    width: 100%;
    background-color: rgb(255, 255, 255);
    padding: 1rem 0;
    text-align: center;
    font-size: 0.875rem;
    border-top: 1px solid var(--border-color);
  }

  .container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    max-width: 1000px;
    margin: 0 auto;
  }

  .footer-links a {
    color: var(--primary-color);
    margin-left: 15px;
    text-decoration: none;
  }

  .footer-links a:hover {
    text-decoration: underline;
    color: #ffdd57;
  }
</style>
