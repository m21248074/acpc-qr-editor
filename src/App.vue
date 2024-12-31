<script setup>
import { reactive, ref, watch } from 'vue';
import { RouterLink, RouterView } from 'vue-router'

import jsQR from "jsqr";
import QRCode from 'qrcode';

const canvas = ref();

const json = reactive({
  "Version": "",
  "PlayerId": "",
  "PlayerName": "",
  "PartnerId": "",
  "Sex": "",
  "EyeParts": "",
  "HairStyle": "",
  "EyeColorIndex": "",
  "HairColorIndex": "",
  "SkinColorIndex": "",
  "PlayerClothes": "",
  "PartnerClothes": "",
  "BGColor": "",
  "Posing": "",
  "Level": "",
  "PlayerMyDesignId": "",
  "PartnerMyDesignId": ""
})

let prefixBinary;
let postfixBinary;

// Helper: Convert string to binary (UTF-8)
function stringToBinary(string) {
  const encoder = new TextEncoder();
  let r = encoder.encode(string);
  console.log(`stringToBinary: ${r}`);
  return r;
}

function concatenate(uint8arrays) {
  const totalLength = uint8arrays.reduce(
    (total, uint8array) => total + uint8array.byteLength,
    0
  );
  const result = new Uint8Array(totalLength);
  let offset = 0;
  uint8arrays.forEach(uint8array => {
    result.set(uint8array, offset);
    offset += uint8array.byteLength;
  });
  return result;
}

watch(json, () => {

  const jsonStri = JSON.stringify(json);
  const jsonBinary = stringToBinary(jsonStri);
  let jsonLength = jsonBinary.length;
  console.log(`jsonLength: ${jsonLength}`)
  let byte1 = (jsonLength % 128) + 128;
  let byte2 = Math.floor((jsonLength - 128) / 128) + 1;
  console.log('length bytes');
  console.log(byte1);
  console.log(byte2);
  prefixBinary[prefixBinary.length - 2] = byte1;
  prefixBinary[prefixBinary.length - 1] = byte2;
  console.log(prefixBinary);
  const binaryData = concatenate([
    prefixBinary,
    jsonBinary,
    postfixBinary,
  ]);

  let cArray = new Uint8ClampedArray(binaryData);

  // Generate a new QR code
  QRCode.toCanvas(
    canvas.value,
    [{ data: cArray, mode: 'byte' }], //st
    { version: 23 }, //{ width: 800, scale:6 }
    (error) => {
      if (error) {
        console.error(error);
        return;
      }
    }
  );

})

const processQR = (ctx) => {
  const imageData = ctx.getImageData(0, 0, canvas.value.width, canvas.value.height);
  const qrCodeData = jsQR(imageData.data, canvas.value.width, canvas.value.height);

  let uint8Array = new Uint8Array(qrCodeData.binaryData);
  console.log(uint8Array);
  console.log(uint8Array.length);

  let rawData = new TextDecoder('utf-8').decode(uint8Array);
  console.log(rawData);
  console.log(rawData.length);

  const strData = rawData.substring(24, rawData.length - 1);

  prefixBinary = uint8Array.slice(0, 24);
  postfixBinary = uint8Array.slice(uint8Array.length - 1);

  let jsonObj = JSON.parse(strData);
  console.log('json length: ', strData.length);

  console.log('text:', strData);
  console.log('prefix binary:', prefixBinary);
  console.log('postfix binary:', postfixBinary);

  for (let [k, v] of Object.entries(jsonObj))
    json[k] = v;
}

const imageToCanvas = (file) => {
  const ctx = canvas.value.getContext('2d', { willReadFrequently: true });
  const img = new Image();
  img.addEventListener('load', () => {
    canvas.value.width = img.width;
    canvas.value.height = img.height;
    ctx.drawImage(img, 0, 0);
    processQR(ctx);
  });
  img.src = URL.createObjectURL(file);
}

const uploadImage = (e) => {
  const file = e.target.files[0];
  imageToCanvas(file);
}

const init = async () => {
  const response = await fetch(`baseQRCode.png`);
  // here image is url/location of image
  const blob = await response.blob();
  const file = new File([blob], 'image.jpg', { type: blob.type });
  imageToCanvas(file);
}

init();

</script>

<template>
  <div class="container-fluid">

    <div class="modal modal-xl d-block"
      :style="{ backgroundImage: 'url(background.jpg)', backgroundSize: 'cover', backgroundRepeat: `no-repeat`, backgroundPosition: 'center' }"
      tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content" :style="{ backgroundColor: `rgba(255, 255, 255, 0.8)` }">

          <div class="modal-body text-center">
            <h2 class="fw-bold">動物森友會 口袋露營廣場 - 非官方露營者卡片編輯器</h2>
            <h4 class="text-body-secondary">編輯您的露營者卡片 QR Code 以導入自訂露營者!</h4>

            <div class="text-start">
              <h5 class="fw-bold">注意事項</h5>
              <p>若要在遊戲中匯入您自己，您必須變更玩家ID。</p>
              <p>
                <b>背景顏色</b>: 0 - 橘色 1 - 粉色 2 - 淺綠色 3 - 深綠色<br />
                <span class="invisible">背景顏色:</span> 4 - 棕色 5 - 紅色 6 - 紫色 7 - 藍色
              </p>
              <p><b>玩家名稱</b>: 最多10字元</p>
              <p><b>最大等級</b>: 1999999999</p>
            </div>

            <div class="row">
              <div class="col-5">
                <input type="file" class="form-control" @change="uploadImage" />
                <canvas ref="canvas" :style="{ width: `468px`, height: `468px` }"></canvas>
              </div>
              <div class="col">
                <div class="input-group mb-3">
                  <span class="input-group-text" title="版本號，預設為1">Version</span>
                  <input type="text" class="form-control" v-model="json['Version']">
                  <span class="input-group-text" title="玩家ID">PlayerId</span>
                  <input type="text" class="form-control" v-model="json['PlayerId']">
                </div>
                <div class="input-group mb-3">
                  <span class="input-group-text" title="玩家名稱">PlayerName</span>
                  <input type="text" class="form-control" v-model="json['PlayerName']">
                  <span class="input-group-text" title="夥伴名稱">PartnerId</span>
                  <input type="text" class="form-control" v-model="json['PartnerId']">
                  <span class="input-group-text" title="性別">Sex</span>
                  <input type="text" class="form-control" v-model="json['Sex']">
                </div>
                <div class="input-group mb-3">
                  <span class="input-group-text" title="眼睛類型">EyeParts</span>
                  <input type="text" class="form-control" v-model="json['EyeParts']">
                  <span class="input-group-text" title="髮型">HairStyle</span>
                  <input type="text" class="form-control" v-model="json['HairStyle']">
                </div>
                <div class="input-group mb-3">
                  <span class="input-group-text" title="眼睛顏色">EyeColorIndex</span>
                  <input type="text" class="form-control" v-model="json['EyeColorIndex']">
                  <span class="input-group-text" title="頭髮顏色">HairColorIndex</span>
                  <input type="text" class="form-control" v-model="json['HairColorIndex']">
                  <span class="input-group-text" title="皮膚顏色">SkinColorIndex</span>
                  <input type="text" class="form-control" v-model="json['SkinColorIndex']">
                </div>
                <div class="input-group mb-3">
                  <span class="input-group-text" title="玩家服裝">PlayerClothes</span>
                  <input type="text" class="form-control" v-model="json['PlayerClothes']">
                </div>
                <div class="input-group mb-3">
                  <span class="input-group-text" title="夥伴服裝">PartnerClothes</span>
                  <input type="text" class="form-control" v-model="json['PartnerClothes']">
                </div>
                <div class="input-group mb-3">
                  <span class="input-group-text" title="背景顏色">BGColor</span>
                  <input type="text" class="form-control" v-model="json['BGColor']">
                  <span class="input-group-text" title="姿勢">Posing</span>
                  <input type="text" class="form-control" v-model="json['Posing']">
                  <span class="input-group-text" title="等級">Level</span>
                  <input type="text" class="form-control" v-model="json['Level']">
                </div>
                <div class="input-group mb-3">
                  <span class="input-group-text" title="玩家穿著設計服裝">PlayerMyDesignId</span>
                  <input type="text" class="form-control" v-model="json['PlayerMyDesignId']">
                  <span class="input-group-text" title="夥伴穿著設計服裝">PartnerMyDesignId</span>
                  <input type="text" class="form-control" v-model="json['PartnerMyDesignId']">
                </div>
              </div>
            </div>
          </div>

        </div>
      </div>
    </div>

  </div>
</template>
