<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>SCANNER</ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">SCANNER</ion-title>
        </ion-toolbar>
      </ion-header>

      <div id="container">
        {{ result }}
        <button class="btn" @click="startScanner()">SCAN</button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar } from '@ionic/vue';
import { defineComponent } from 'vue';
import { BarcodeScanner } from '@ionic-native/barcode-scanner';

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
      result: null,
      isFlashOn: false,
    }
  },
  methods: {
    async startScanner() {
      BarcodeScanner.scan({
        prompt : "Place a barcode inside the scan area, :)", // Android
        showFlipCameraButton : true,
        showTorchButton : true,
        disableAnimations : true,
        disableSuccessBeep: false
      }).then(barcodeData => {
        console.log('Barcode data', barcodeData)
        this.result = barcodeData;
      }).catch(err => {
        console.log('Error', err)
      });
    },
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