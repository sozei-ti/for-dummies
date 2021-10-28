# Deploy App (Play Store)

## Preparando o App
<br />

### Alterar a versão do projeto
Com projeto aberto em seu editor, acesse: `android/app/build.gradle`, e procure o defaultConfig para atualizarmos a versão do projeto.

```js
defaultConfig {
        applicationId "com.sozei.regallo"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 28 //update here
        versionName "1.28" // update here
        vectorDrawables.useSupportLibrary = true
    }
```

### Gerando APK

Agora você precisa fazer um PR para mergear essas alterações em sua branch main.
Assim que o merge na main for concluído a action responsável por gerar o apk vai rodar, e quando ela finalizar só baixar o zip fixado.

<br />

## Subindo na Loja
Todos os passos abaixos devem ser feitos no
[Google play console](https://play.google.com/console/).

Acesse:
Todos os apps / {App desejado} / Produção/ Criar nova versão

Agora você precisa fazer o upload do arquivo AAB que está dentro do zip que você baixou do github actions.

Depois:
Salvar / Avaliar versão / Iniciar lançamento para produção.

A partir desse momento seu app vai ficar em avaliação pela google play.