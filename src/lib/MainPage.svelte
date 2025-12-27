<script>
    export let token = "";

    let scanResult = "";
    let isPayLoading = false;

    function handleScan() {
        if (typeof my !== "undefined") {
            my.scan({
                type: "qr",
                success: (res) => {
                    // Result handling: res.code is the QR content
                    scanResult = res.code || res.qrCode || "No data";
                },
                fail: (res) => {
                    my.alert({
                        content: "Scan failed: " + JSON.stringify(res),
                    });
                },
            });
        } else {
            // Mock for dev
            scanResult = "Parking-Area-123";
            alert("Dev Mode: Scanned mock QR code");
        }
    }

    function handlePay() {
        if (!token) {
            alert("Missing auth token. Please relogin.");
            return;
        }

        isPayLoading = true;

        // Use the exact fetch logic provided
        fetch("https://its.mouamle.space/api/payment", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                Authorization: token,
            },
        })
            .then((res) => res.json())
            .then((data) => {
                isPayLoading = false;

                if (typeof my !== "undefined") {
                    my.tradePay({
                        paymentUrl: data.url,
                        success: (res) => {
                            my.alert({
                                content: "Payment successful",
                            });
                            // Reset after success
                            scanResult = "";
                        },
                        fail: (res) => {
                            my.alert({
                                content: "Payment cancelled or failed",
                            });
                        },
                    });
                } else {
                    console.log("Dev Mode: Payment URL received", data.url);
                    alert("Dev Mode: TradePay triggered. URL: " + data.url);
                }
            })
            .catch((err) => {
                isPayLoading = false;
                let errorDetails = String(err);
                if (typeof my !== "undefined") {
                    my.alert({
                        content:
                            "Payment initialization failed: " + errorDetails,
                    });
                } else {
                    console.error(err);
                    alert("Payment initialization failed");
                }
            });
    }
</script>

<div class="min-h-screen bg-gray-50 flex flex-col items-center p-6 relative">
    <!-- Header -->
    <div
        class="absolute top-0 left-0 w-full h-16 bg-gradient-to-r from-blue-600 to-blue-800 shadow-md flex items-center px-6 z-10"
    >
        <h1 class="text-xl font-bold text-white tracking-wide">
            Parking Payment
        </h1>
    </div>

    <div class="w-full max-w-md mt-20 flex flex-col gap-6">
        <!-- Scan Section -->
        <div class="bg-white rounded-2xl shadow-lg p-6 text-center">
            <div class="mb-4 flex justify-center">
                <div class="bg-blue-50 p-4 rounded-full">
                    <svg
                        xmlns="http://www.w3.org/2000/svg"
                        class="h-12 w-12 text-blue-600"
                        fill="none"
                        viewBox="0 0 24 24"
                        stroke="currentColor"
                    >
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                            d="M12 4v1m6 11h2m-6 0h-2v4h-4v-4H8m13-4V4H4v16h16v-9"
                        />
                    </svg>
                </div>
            </div>
            <h2 class="text-xl font-bold text-gray-800 mb-2">Scan & Pay</h2>
            <p class="text-gray-500 mb-6 text-sm">
                Scan the parking QR code to proceed with payment.
            </p>

            <button
                class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 px-6 rounded-xl shadow-md transition transform active:scale-95 flex items-center justify-center gap-2"
                onclick={handleScan}
            >
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-6 w-6"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"
                    />
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M15 13a3 3 0 11-6 0 3 3 0 016 0z"
                    />
                </svg>
                Scan QR Code
            </button>
        </div>

        <!-- Result & Pay Section -->
        {#if scanResult}
            <div
                class="bg-white rounded-2xl shadow-lg p-6 border-l-4 border-green-500 animate-fade-in-up"
            >
                <h3 class="text-lg font-bold text-gray-800 mb-2">
                    Selected Spot
                </h3>
                <div
                    class="bg-gray-50 p-3 rounded-lg border border-gray-100 mb-4"
                >
                    <p class="font-mono text-gray-600 text-lg break-all">
                        {scanResult}
                    </p>
                </div>

                <button
                    class="w-full {isPayLoading
                        ? 'bg-gray-400 cursor-not-allowed'
                        : 'bg-green-600 hover:bg-green-700'} text-white font-bold py-3 px-6 rounded-xl shadow-md transition transform active:scale-95 flex items-center justify-center gap-2"
                    onclick={handlePay}
                    disabled={isPayLoading}
                >
                    {#if isPayLoading}
                        <svg
                            class="animate-spin h-5 w-5 text-white"
                            xmlns="http://www.w3.org/2000/svg"
                            fill="none"
                            viewBox="0 0 24 24"
                        >
                            <circle
                                class="opacity-25"
                                cx="12"
                                cy="12"
                                r="10"
                                stroke="currentColor"
                                stroke-width="4"
                            ></circle>
                            <path
                                class="opacity-75"
                                fill="currentColor"
                                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                            ></path>
                        </svg>
                        Processing...
                    {:else}
                        <svg
                            xmlns="http://www.w3.org/2000/svg"
                            class="h-6 w-6"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor"
                        >
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                stroke-width="2"
                                d="M3 10h18M7 15h1m4 0h1m-7 4h12a3 3 0 003-3V8a3 3 0 00-3-3H6a3 3 0 00-3 3v8a3 3 0 003 3z"
                            />
                        </svg>
                        Pay Now
                    {/if}
                </button>
            </div>
        {/if}
    </div>
</div>

<style>
    @keyframes fadeInUp {
        from {
            opacity: 0;
            transform: translateY(20px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }
    .animate-fade-in-up {
        animation: fadeInUp 0.4s ease-out forwards;
    }
</style>
