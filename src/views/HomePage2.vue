<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>SCANNING</ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">SCANNING</ion-title>
        </ion-toolbar>
      </ion-header>

      <div id="container">
        {{ result }}
        <button v-if="!scanning" class="btn" @click="startScanner()">SCAN</button>
        <button v-if="scanning" class="flash-btn" :class="{ 'flash-on': isFlashOn }" @click="flashOn()">FLASH</button>
        <button v-if="scanning" class="btn" @click="stopScanner()">STOP</button>
        <button v-if="permissionDenied">DENIED</button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, alertController } from '@ionic/vue';
import { defineComponent } from 'vue';
import { BarcodeScanner } from '@capacitor-community/barcode-scanner';
import { Flashlight } from '@awesome-cordova-plugins/flashlight';

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar
  },
  data() {
    return {
      permissionDenied: false,
      scanning: false,
      result: null,
      isFlashOn: false
    }
  },
  methods: {
    async flashOn() {
      await Flashlight.toggle()
      this.isFlashOn = true
    },
    async startScanner() {
      if (await this.didUserGrantPermissions()) {
        this.permissionDenied = false
        this.scanning = true
        BarcodeScanner.hideBackground() // make background of WebView transparent
  
        const result = await BarcodeScanner.startScan({
          torchOn: true,
          showTorchButton : true,
        }) // start scanning and wait for a result
  
        // if the result has content
        if (result.hasContent) {
          this.stopScanner()
          this.result = result.content // log the raw scanned content
        }

        document.body.classList.add('camera-hide')
      } else {
        this.permissionDenied = true
      }
    },
    stopScanner() {
      this.scanning = false
      BarcodeScanner.showBackground()
      BarcodeScanner.stopScan()
      document.body.classList.remove('camera-hide')
    },
    async didUserGrantPermissions() {
      // check if user already granted permission
      const status = await BarcodeScanner.checkPermission({ force: false });

      if (status.granted) {
        // user granted permission
        return true;
      }

      if (status.denied) {
        // user denied permission
        return false;
      }

      if (status.asked) {
        // system requested the user for permission during this call
        // only possible when force set to true
      }

      if (status.neverAsked) {
        // user has not been requested this permission before
        // it is advised to show the user some sort of prompt
        // this way you will not waste your only chance to ask for the permission
        const c = confirm(
          'We need your permission to use your camera to be able to scan barcodes',
        );
        if (!c) {
          return false;
        }
      }

      if (status.restricted || status.unknown) {
        // ios only
        // probably means the permission has been denied
        return false;
      }

      // user has not denied permission
      // but the user also has not yet granted the permission
      // so request it
      const statusRequest = await BarcodeScanner.checkPermission({ force: true });

      if (statusRequest.asked) {
        // system requested the user for permission during this call
        // only possible when force set to true
      }

      if (statusRequest.granted) {
        // the user did grant the permission now
        return true;
      }

      // user did not grant the permission, so he must have declined the request
      return false;
    },
    async checkPermission() {
      return new Promise((resolve) => {
        const status = BarcodeScanner.checkPermission({ force: true });

        if (status.granted) {
          resolve(true);
        } else if (status.denied) {
          alertController.create({
            header: 'No permission',
            message: 'Please allow camera acccess in your settings',
            buttons: [
              {
                text: 'No',
                role: 'cancel'
              },
              {
                text: 'Open Settings',
                handler() {
                  BarcodeScanner.openAppSettings();
                  resolve(false);
                }
              }
            ]
          })
        } else {
          resolve(false);
        }
      })
    }
  }
});
</script>

<style>
html, body {
  background: transparent;
}

ion-content {
  --background: transparent;
}

.flash-btn {
  padding: 20px;
}

.flash-on {
  background: #000;
  color: #fff;
}

#container {
  text-align: center;
  
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}

.btn {
  padding: 50px 100px;
}
</style>