# Animal Crossing: Pocket Camp Complete - Unofficial Camper Card Editor

## 動物森友會: 口袋露營廣場 - 非官方露營者卡片編輯器

How to Use:
 - go to https://m21248074.github.io/acpc-qr-editor/
 - upload a camper card file
 - make changes to the input fields
 - import the new camper card by

Notes:
 - There's an issue with reading certain camper codes. If this app isn't reading the QR you saved from AC:PC, try changing the background colour. You can change the colour back once imported.
 - To import copies of yourself, you must change the PlayerId value.
 - BGColor: 0-orange 1-pink 2-lightgreen 3-darkgreen 4-brown 5-red 6-purple 7-blue
 - PlayerName: max 10 characters
 - max Level: 1999999999

Partners:
 - set with PartnerId field

| Name           | PartnerId      |
|----------------|----------------|
| Roald          | 224800         |

Clothes:
 - set with the PlayerClothes and PartnerClothes fields
 - keep in mind that some clothes only work on players

| Name       | ID       | index (ltr - 1,2,..)       |
|----------------|----------------|----------------|
| mint gingham tee  | 216  | 3  |
| red snowperson outfit  | 5075083  | 3  |

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```