<script>
  import AuthPage from "./lib/authPage.svelte";
  import MainPage from "./lib/MainPage.svelte";

  let token = "";
  let isLoggedIn = false;

  function handleLoginSuccess(newToken) {
    if (newToken) {
      token = newToken;
      isLoggedIn = true;
    } else {
      // Fallback or error handling if token is missing in response
      alert("Login successful but no token received!");
    }
  }
</script>

<main>
  {#if !isLoggedIn}
    <AuthPage onLoginSuccess={handleLoginSuccess} />
  {:else}
    <MainPage {token} />
  {/if}
</main>
