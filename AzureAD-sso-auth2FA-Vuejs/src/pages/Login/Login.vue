<template>
  <div id="app" class="login-container" :style="{ backgroundImage: 'url(' + currentImage + ')' }">
    <div class="login-panel">
      <h1>Login</h1>
      <button @click="iniciarAutenticacao">Entrar com Azure AD</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      images: [
        'https://images.pexels.com/photos/4484078/pexels-photo-4484078.jpeg',
        'https://images.pexels.com/photos/4487383/pexels-photo-4487383.jpeg',
        'https://images.pexels.com/photos/4481262/pexels-photo-4481262.jpeg',
      ],
      currentImageIndex: 0,
    };
  },
  computed: {
    currentImage() {
      return this.images[this.currentImageIndex];
    },
  },
  created() {
    setInterval(() => {
      this.currentImageIndex = (this.currentImageIndex + 1) % this.images.length;
    }, 5000);
  },
  methods: {
    iniciarAutenticacao() {
      this.openPopup();
    },
    openPopup() {
      var token = "";
      const currentUri = encodeURIComponent(window.location.origin);
      //Caminho da API do Servidor AD deixei como exemplo : porem você precisa substituir https://login.microsoftonline.com/sso/login
      const popupUrl = `https://login.microsoftonline.com/?redirect_uri=${currentUri}`;
      const width = 600;
      const height = 600;
      const left = (window.innerWidth - width) / 2;
      const top = (window.innerHeight - height) / 2;
      const popupWindow = window.open(popupUrl, 'samlLogin', `width=${width},height=${height},left=${left},top=${top}`);

      const handleMessage = (event) => {
        let data;
        if (typeof event.data === "string") {
          try {
            clearInterval(checkPopupClosed);
            data = JSON.parse(event.data);
            console.log("Dados após o parse: ", data.result.auth_token);
            token = data.result.auth_token;
            if (token) {
              this.redirecionarParaDashboard();
            }
          } catch (error) {
            return;
          }
        } else {
          data = event.data;
        }
        window.removeEventListener('message', handleMessage);
      }
      window.addEventListener('message', handleMessage);

      const checkPopupClosed = setInterval(() => {
        if (popupWindow.closed) {
          clearInterval(checkPopupClosed);
          window.removeEventListener('message', handleMessage);
          if (token == '') {
            alert("Não autenticado.");
          }
        }
      }, 500);
    },
    redirecionarParaDashboard() {
      this.$router.push('/dashboard');
    },
  },
};
</script>

<style>
#app {
  height: 100vh;
  background-size: cover;
  background-position: center;
  transition: background-image 1s ease-in-out;
  animation: fadeIn 2s;
}

.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.login-panel {
  text-align: center;
  padding: 20px;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

h1 {
  font-size: 2em;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #0078d4;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
</style>
