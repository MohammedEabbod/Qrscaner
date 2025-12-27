<script>
    let authCode = "";
    let statusMessage = "";
    let isError = false;

    function authenticate() {
        statusMessage = "Requesting access...";
        isError = false;

        // Check if 'my' object exists (Hylid environment)
        if (typeof my !== "undefined") {
            my.getAuthCode({
                scopes: ["auth_base", "USER_ID"],
                success: (res) => {
                    authCode = res.authCode;
                    statusMessage = "Auth code received. Verifying...";

                    // Send to backend
                    fetch("https://its.mouamle.space/api/auth-with-superQi", {
                        method: "POST",
                        headers: {
                            "Content-Type": "application/json",
                        },
                        body: JSON.stringify({
                            token: authCode,
                        }),
                    })
                        .then((res) => res.json())
                        .then((data) => {
                            statusMessage = "Login successful!";
                            my.alert({
                                content: "Login successful",
                            });
                        })
                        .catch((err) => {
                            isError = true;
                            let errorDetails = "";
                            if (err && typeof err === "object") {
                                errorDetails = JSON.stringify(err, null, 2);
                            } else {
                                errorDetails = String(err);
                            }
                            statusMessage = "Error: " + errorDetails;
                            my.alert({
                                content: "Error: " + errorDetails,
                            });
                        });
                },
                fail: (res) => {
                    isError = true;
                    // Log the full error to help debug
                    console.error("Auth failed:", res);
                    statusMessage =
                        "Authorization failed: " + JSON.stringify(res, null, 2);
                },
            });
        } else {
            // Fallback for development outside Super Qi app
            console.warn(
                "'my' object is undefined. Are you running in the Super Qi app?",
            );
            isError = true;
            statusMessage =
                "Environment not supported. Please open in Super Qi App.";

            // Mock for dev
            // authCode = "mock_auth_code_123";
            // statusMessage = "Mock Auth Code Generated"
        }
    }

    function copyAuthCode() {
        if (authCode) {
            navigator.clipboard.writeText(authCode);
            if (typeof my !== "undefined") {
                my.alert({ content: "Copied to clipboard" });
            } else {
                alert("Copied to clipboard");
            }
        }
    }
</script>

<div
    class="min-h-screen bg-gray-50 flex flex-col items-center justify-center p-6 relative"
>
    <!-- Header/Nav -->
    <div
        class="absolute top-0 left-0 w-full h-16 bg-gradient-to-r from-blue-600 to-blue-800 shadow-md flex items-center px-6"
    >
        <h1 class="text-xl font-bold text-white tracking-wide">Demo MiniApp</h1>
    </div>

    <!-- Main Card -->
    <div
        class="w-full max-w-md bg-white rounded-2xl shadow-xl overflow-hidden mt-16 transform transition-all hover:scale-[1.01] duration-300"
    >
        <div class="p-8 flex flex-col items-center text-center">
            <div
                class="h-20 w-20 bg-blue-100 rounded-full flex items-center justify-center mb-6"
            >
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-10 w-10 text-blue-600"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"
                    />
                </svg>
            </div>

            <h2 class="text-2xl font-bold text-gray-800 mb-2">
                Authentication Required
            </h2>
            <p class="text-gray-500 mb-8">
                Please log in to access the full features of the MiniApp.
            </p>

            <button
                class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-6 rounded-xl shadow-lg transition duration-200 transform active:scale-95 mb-4 flex items-center justify-center gap-2 cursor-pointer"
                onclick={authenticate}
            >
                <span>Login with Super Qi</span>
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-5 w-5"
                    viewBox="0 0 20 20"
                    fill="currentColor"
                >
                    <path
                        fill-rule="evenodd"
                        d="M3 3a1 1 0 011 1v12a1 1 0 11-2 0V4a1 1 0 011-1zm7.707 3.293a1 1 0 010 1.414L9.414 9H17a1 1 0 110 2H9.414l1.293 1.293a1 1 0 01-1.414 1.414l-3-3a1 1 0 010-1.414l3-3a1 1 0 011.414 0z"
                        clip-rule="evenodd"
                    />
                </svg>
            </button>

            {#if authCode}
                <button
                    class="w-full bg-gray-100 hover:bg-gray-200 text-gray-700 font-medium py-3 px-6 rounded-xl transition duration-200 flex items-center justify-center gap-2 cursor-pointer"
                    onclick={copyAuthCode}
                >
                    <svg
                        xmlns="http://www.w3.org/2000/svg"
                        class="h-5 w-5"
                        fill="none"
                        viewBox="0 0 24 24"
                        stroke="currentColor"
                    >
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                            d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"
                        />
                    </svg>
                    Copy Auth Code
                </button>
                <div
                    class="mt-4 p-3 bg-gray-50 rounded-lg w-full text-left border border-gray-100"
                >
                    <p
                        class="text-xs text-gray-400 uppercase font-bold tracking-wider mb-1"
                    >
                        Auth Code
                    </p>
                    <p
                        class="font-mono text-sm text-gray-600 truncate bg-white p-2 rounded border border-gray-200 select-all"
                    >
                        {authCode}
                    </p>
                </div>
            {/if}

            {#if statusMessage}
                <div
                    class="mt-6 p-4 rounded-lg w-full {isError
                        ? 'bg-red-50 text-red-600 border border-red-100'
                        : 'bg-green-50 text-green-600 border border-green-100'} animate-fade-in"
                >
                    <p class="text-sm font-medium">{statusMessage}</p>
                </div>
            {/if}
        </div>
    </div>

    <div class="mt-8 text-center text-gray-400 text-xs">
        <p>&copy; 2024 demo MiniApp. Safe & Secure.</p>
    </div>
</div>

<style>
    /* Slight animation for the status message */
    @keyframes fadeIn {
        from {
            opacity: 0;
            transform: translateY(10px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }
    .animate-fade-in {
        animation: fadeIn 0.3s ease-out forwards;
    }
</style>
