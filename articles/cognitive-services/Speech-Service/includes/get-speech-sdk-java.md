---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: fcc04cc4599681e3542b275d238b75352d690a95
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2020
ms.locfileid: "93135823"
---
:::row:::
    :::column span="3":::
        El SDK de Java para Android está empaquetado como una <a href="https://developer.android.com/studio/projects/android-library" target="_blank">biblioteca de Android (AAR)<span class="docon docon-navigate-external x-hidden-focus"></span></a>, que incluye las bibliotecas necesarias, así como los permisos necesarios de Android. Se hospeda en un repositorio de Maven en `https://csspeechstorage.blob.core.windows.net/maven/` como un paquete `com.microsoft.cognitiveservices.speech:client-sdk:1.14.0`.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Java" src="https://docs.microsoft.com/media/logos/logo_java.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

Para consumir el paquete desde el proyecto de Android Studio, haga los siguientes cambios:

1. En el archivo *build.gradle* de nivel de proyecto, agregue lo siguiente a la sección `repositories`:
  ```gradle
  maven { url 'https://csspeechstorage.blob.core.windows.net/maven/' }
  ```

2. En el archivo *build.gradle* de nivel de módulo, agregue lo siguiente a la sección `dependencies`:
  ```gradle
  implementation 'com.microsoft.cognitiveservices.speech:client-sdk:1.14.0'
  ```

El SDK de Java es parte del [SDK de dispositivos de voz](../speech-devices-sdk.md).

#### <a name="additional-resources"></a>Recursos adicionales

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android" target="_blank">Código fuente de la guía de inicio rápido de Java específico para Android <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/jre" target="_blank">Código fuente dela guía de inicio rápido de Java Runtime (JRE) <span class="docon docon-navigate-external x-hidden-focus"></span></a>
