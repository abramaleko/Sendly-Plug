# Sendy-Plug

Plug In for subcribe/unsubscribe of single/multiple email users for Sendy 

## How To Use
This plugin is simple to use for setting up the project make sure you have node installed in your computer and you can follow the project setup process below

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

## Deploying on Sub directory
If you are deploying this project to a sub directory make sure to set up the base path for your project, So that it can load assets from your base path. Locate the vite.config.js file and add it there. 

If your uploading to main dir you can set it to '/' or just remove the the base variable 

```sh
export default defineConfig({
  base: '/my-subdirectory/',
  plugins: [
    vue(),
  ],
  ....
```

After doing this make sure to build your project again using

```sh
npm run build
```

## Setting Enviroment Variables
The format for the enviroment variable are already defined in the .env.example file make sure you set them up before building your app.

### Pre-Loading Brands
If you want the plugin to only access only one specific brand you can configure them in the .env file. If not you can just comment them out using '#' and will load all the brands

```
VITE_BRAND_ID=
VITE_BRAND_NAME=
```