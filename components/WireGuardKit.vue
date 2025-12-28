<template>
  <div
    class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0"
  >
    <link
      href="https://cdn.jsdelivr.net/npm/tailwindcss@2.1.2/dist/tailwind.min.css"
      rel="stylesheet"
    />
    <div class="max-w-4xl mx-auto sm:px-6 lg:px-8 py-8">
      <div class="bg-white overflow-hidden shadow sm:rounded-lg p-6 mb-6">
        <h1 class="text-3xl leading-9 font-bold text-gray-900 mb-4">
          WireGuard iOS Configuration Kit
        </h1>
        <p class="text-gray-600 mb-4">
          Configure your custom WireGuard VPN settings for iOS devices. This
          tool helps you generate and manage WireGuard configurations optimized
          for iOS.
        </p>
      </div>

      <div class="bg-white overflow-hidden shadow sm:rounded-lg p-6 mb-6">
        <h2 class="text-2xl leading-7 font-semibold mb-4">
          Configuration Settings
        </h2>
        <form class="space-y-4" @submit.prevent="generateConfig">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Interface Name
            </label>
            <input
              v-model="config.interfaceName"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
              placeholder="e.g., wg0"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Private Key
            </label>
            <input
              v-model="config.privateKey"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
              placeholder="Your private key"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Address (IPv4/IPv6)
            </label>
            <input
              v-model="config.address"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
              placeholder="e.g., 10.0.0.2/24"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">
              DNS Servers
            </label>
            <input
              v-model="config.dns"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
              placeholder="e.g., 1.1.1.1, 8.8.8.8"
            />
          </div>

          <div class="border-t border-gray-200 pt-4 mt-4">
            <h3 class="text-lg font-semibold mb-3">Peer Configuration</h3>

            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">
                  Public Key
                </label>
                <input
                  v-model="config.peerPublicKey"
                  type="text"
                  class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
                  placeholder="Peer's public key"
                />
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">
                  Endpoint
                </label>
                <input
                  v-model="config.endpoint"
                  type="text"
                  class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
                  placeholder="e.g., vpn.example.com:51820"
                />
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">
                  Allowed IPs
                </label>
                <input
                  v-model="config.allowedIPs"
                  type="text"
                  class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
                  placeholder="e.g., 0.0.0.0/0, ::/0"
                />
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">
                  Persistent Keepalive (seconds)
                </label>
                <input
                  v-model="config.persistentKeepalive"
                  type="number"
                  class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-green-500"
                  placeholder="25"
                />
              </div>
            </div>
          </div>

          <div class="flex space-x-4 pt-4">
            <button
              type="submit"
              class="px-6 py-2 bg-green-500 text-white rounded-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2"
            >
              Generate Configuration
            </button>
            <button
              type="button"
              class="px-6 py-2 bg-gray-500 text-white rounded-md hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
              @click="resetConfig"
            >
              Reset
            </button>
          </div>
        </form>
      </div>

      <div
        v-if="generatedConfig"
        class="bg-white overflow-hidden shadow sm:rounded-lg p-6 mb-6"
      >
        <h2 class="text-2xl leading-7 font-semibold mb-4">
          Generated Configuration
        </h2>
        <div class="bg-gray-900 text-green-400 p-4 rounded-md overflow-x-auto">
          <pre class="text-sm">{{ generatedConfig }}</pre>
        </div>
        <div class="mt-4 flex space-x-4">
          <button
            class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
            @click="copyToClipboard"
          >
            Copy to Clipboard
          </button>
          <button
            class="px-4 py-2 bg-purple-500 text-white rounded-md hover:bg-purple-600 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:ring-offset-2"
            @click="downloadConfig"
          >
            Download .conf File
          </button>
        </div>
        <p v-if="copySuccess" class="mt-2 text-green-600 text-sm">
          Copied to clipboard!
        </p>
      </div>

      <div class="bg-white overflow-hidden shadow sm:rounded-lg p-6">
        <h2 class="text-2xl leading-7 font-semibold mb-4">
          iOS Installation Guide
        </h2>
        <ol class="list-decimal list-inside space-y-2 text-gray-700">
          <li>Install the WireGuard app from the iOS App Store</li>
          <li>Open the WireGuard app on your iOS device</li>
          <li>Tap the "+" button to add a new tunnel</li>
          <li>Choose "Create from file or archive" or "Create from QR code"</li>
          <li>Import your generated configuration file</li>
          <li>Toggle the connection to activate your VPN</li>
        </ol>
        <div class="mt-4 p-4 bg-blue-50 border-l-4 border-blue-500">
          <p class="text-sm text-blue-700">
            <strong>Note:</strong> For iOS, WireGuard provides a native app that
            offers superior performance and battery efficiency compared to other
            VPN protocols. Make sure your iOS device is running iOS 12.0 or
            later.
          </p>
        </div>
      </div>

      <div class="flex justify-center pt-6">
        <a
          href="/"
          class="px-6 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-400 focus:ring-offset-2"
        >
          Back to Home
        </a>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WireGuardKit',
  data() {
    return {
      config: {
        interfaceName: 'wg0',
        privateKey: '',
        address: '',
        dns: '1.1.1.1, 8.8.8.8',
        peerPublicKey: '',
        endpoint: '',
        allowedIPs: '0.0.0.0/0, ::/0',
        persistentKeepalive: 25,
      },
      generatedConfig: '',
      copySuccess: false,
    }
  },
  methods: {
    generateConfig() {
      const {
        interfaceName,
        privateKey,
        address,
        dns,
        peerPublicKey,
        endpoint,
        allowedIPs,
        persistentKeepalive,
      } = this.config

      if (!privateKey || !address || !peerPublicKey || !endpoint) {
        alert('Please fill in all required fields')
        return
      }

      this.generatedConfig = `[Interface]
# Name: ${interfaceName}
PrivateKey = ${privateKey}
Address = ${address}
DNS = ${dns}

[Peer]
PublicKey = ${peerPublicKey}
Endpoint = ${endpoint}
AllowedIPs = ${allowedIPs}
PersistentKeepalive = ${persistentKeepalive}`
    },
    resetConfig() {
      this.config = {
        interfaceName: 'wg0',
        privateKey: '',
        address: '',
        dns: '1.1.1.1, 8.8.8.8',
        peerPublicKey: '',
        endpoint: '',
        allowedIPs: '0.0.0.0/0, ::/0',
        persistentKeepalive: 25,
      }
      this.generatedConfig = ''
      this.copySuccess = false
    },
    copyToClipboard() {
      if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard
          .writeText(this.generatedConfig)
          .then(() => {
            this.copySuccess = true
            setTimeout(() => {
              this.copySuccess = false
            }, 3000)
          })
          .catch((err) => {
            console.error('Failed to copy:', err)
            this.fallbackCopy()
          })
      } else {
        this.fallbackCopy()
      }
    },
    fallbackCopy() {
      const textArea = document.createElement('textarea')
      textArea.value = this.generatedConfig
      textArea.style.position = 'fixed'
      textArea.style.left = '-999999px'
      document.body.appendChild(textArea)
      textArea.select()
      try {
        document.execCommand('copy')
        this.copySuccess = true
        setTimeout(() => {
          this.copySuccess = false
        }, 3000)
      } catch (err) {
        console.error('Failed to copy:', err)
      }
      document.body.removeChild(textArea)
    },
    downloadConfig() {
      const blob = new Blob([this.generatedConfig], {
        type: 'text/plain;charset=utf-8',
      })
      const url = window.URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href = url
      link.download = `${this.config.interfaceName || 'wireguard'}.conf`
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      window.URL.revokeObjectURL(url)
    },
  },
}
</script>
