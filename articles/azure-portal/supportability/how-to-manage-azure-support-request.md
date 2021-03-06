---
title: Administración de una solicitud de soporte técnico de Azure
description: Describe cómo ver solicitudes de soporte técnico, enviar mensajes, cambiar el nivel de gravedad de las solicitudes, compartir información de diagnóstico con el soporte técnico de Azure, volver a abrir una solicitud de soporte técnico cerrada y cargar archivos.
tags: billing
ms.assetid: 86697fdf-3499-4cab-ab3f-10d40d3c1f70
ms.topic: how-to
ms.date: 12/14/2020
ms.openlocfilehash: 8110f87401da1352309fb55615093d49981c754d
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504821"
---
# <a name="manage-an-azure-support-request"></a>Administración de una solicitud de soporte técnico de Azure

Después de [crear una solicitud de soporte técnico de Azure](how-to-create-azure-support-request.md), puede administrarla en [Azure Portal](https://portal.azure.com), como se describe en este artículo. También puede crear y administrar solicitudes mediante programación, con la [API REST de incidencia de soporte técnico de Azure](/rest/api/support).

## <a name="view-support-requests"></a>Ver las solicitudes de soporte técnico

Para ver los detalles y el estado de las solicitudes de soporte técnico, vaya a **Ayuda y soporte técnico** >  **Todas las solicitudes de soporte técnico**.

:::image type="content" source="media/how-to-manage-azure-support-request/all-requests-lower.png" alt-text="Todas las solicitudes de soporte técnico":::

En esta página, puede buscar, filtrar y ordenar solicitudes de soporte técnico. Seleccione una solicitud de soporte técnico para ver los detalles, incluida su gravedad y los mensajes asociados con la solicitud.

## <a name="send-a-message"></a>Envío de un mensaje

1. En la página **Todas las solicitudes de soporte técnico**, seleccione la solicitud de soporte técnico.

1. En la página **Solicitud de soporte técnico**, seleccione **Nuevo mensaje**.

1. Escriba el mensaje y seleccione **Enviar**.

## <a name="change-the-severity-level"></a>Cambiar el nivel de gravedad

> [!NOTE]
> El nivel de gravedad máximo depende de su [plan de soporte técnico](https://azure.microsoft.com/support/plans).
>

1. En la página **Todas las solicitudes de soporte técnico**, seleccione la solicitud de soporte técnico.

1. En la página **Solicitud de soporte técnico**, seleccione **Cambiar**.

    :::image type="content" source="media/how-to-manage-azure-support-request/change-severity.png" alt-text="Cambiar la gravedad de la solicitud de soporte técnico":::

1. Azure Portal muestra una de dos pantallas, en función de si la solicitud ya está asignada a un ingeniero de soporte técnico:

    - Si no se ha asignado la solicitud, verá una pantalla similar a la siguiente. Seleccione un nivel de gravedad nuevo y, a continuación, seleccione **Cambiar**.

        :::image type="content" source="media/how-to-manage-azure-support-request/unassigned-can-change-severity.png" alt-text="Seleccionar un nivel de gravedad nuevo":::

    - Si se ha asignado la solicitud, verá una pantalla similar a la siguiente. Seleccione **Aceptar** y, a continuación, cree un [nuevo mensaje](#send-a-message) para solicitar un cambio en el nivel de gravedad.

        :::image type="content" source="media/how-to-manage-azure-support-request/assigned-cant-change-severity.png" alt-text="No se puede seleccionar un nuevo nivel de gravedad":::

## <a name="share-diagnostic-information-with-azure-support"></a>Compartir información de diagnóstico con el soporte técnico de Azure

Al crear una solicitud de soporte técnico, la opción **Compartir información de diagnóstico** está seleccionada de forma predeterminada. Esto permite al soporte técnico de Azure recopilar [información de diagnóstico](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) de los recursos de Azure:

* No se puede borrar esta opción después de crear una solicitud.

* Si desactivó la opción al crear una solicitud, puede seleccionarla una vez creada la solicitud.

    1. En la página **Todas las solicitudes de soporte técnico**, seleccione la solicitud.
    
    1. En la página **Solicitud de soporte técnico**, seleccione **Conceder permiso** y, a continuación, **Sí** y **Aceptar**.
    
        :::image type="content" source="media/how-to-manage-azure-support-request/grant-permission-manage.png" alt-text="Conceder permisos para la información de diagnóstico":::

## <a name="upload-files"></a>Carga de archivos

Puede usar la opción de carga de archivos para cargar archivos de diagnóstico o cualquier otro archivo que considere pertinente para la solicitud de soporte técnico.

1. En la página **Todas las solicitudes de soporte técnico**, seleccione la solicitud.

1. En la página **Solicitud de soporte técnico**, busque el archivo y seleccione **Cargar**. Repita el proceso si tiene varios archivos.

    :::image type="content" source="media/how-to-manage-azure-support-request/file-upload.png" alt-text="Carga de un archivo":::.

### <a name="file-upload-guidelines"></a>Instrucciones de carga de archivos

Siga estas directrices cuando use la opción de carga de archivos:

* Para proteger su privacidad, no incluya ninguna información personal en su carga.
* El nombre de archivo debe ser de 110 caracteres como máximo.
* No se puede cargar más de un archivo.
* Los archivos no pueden ser superiores a 4 MB.
* Todos los archivos deben tener una extensión de nombre de archivo, por ejemplo, *.docx* o *.xlsx*. En la tabla siguiente se muestran las extensiones de nombre de archivo que se permiten para la carga.

| 0-9, A-C     | D-G   | H-M         | N-P   | R-T      | U-W        | X-Z     |
|-------------|-------|-------------|-------|----------|------------|---------|
| .7z         | .dat  | .har        | .odx  | .rar     | .tdb       | .xlam   |
| .a          | .db   | .hwl        | .oft  | .rdl     | .tdf       | .xlr    |
| .abc        | .DMP  | .ics        | .old  | .rdlc    | .text      | .xls    |
| .adm        | .do_  | .ini        | .one  | .re_     | .thmx      | .xlsb   |
| .aspx       | .doc  | .java       | .osd  | .reg     | .tif       | .xlsm   |
| .ATF        | .docm | .jpg        | .OUT  | .remove  | .trc       | .xlsx   |
| .b          | .docx | .LDF        | .p1   | .ren     | .TTD       | .xlt    |
| .ba_        | .dotm | .letterhead | .pcap | .rename  | .tx_       | .xltx   |
| .bak        | .dotx | .lnk        | .pdb  | .rft     | .txt       | .xml    |
| .bat        | .dtsx | .lo_        | .pdf  | .rpt     | .uccapilog | .xmla   |
| .blg        | .eds  | .log        | .piz  | .rte     | .uccplog   | .xps    |
| .CA_        | .emf  | .lpk        | .pmls | .rtf     | .udcx      | .xsd    |
| .CAB        | .eml  | .manifest   | .png  | .run     | .vb_       | .xsn    |
| .cap        | .emz  | .master     | .potx | .saz     | .vbs_      | .xxx    |
| .catx       | .err  | .mdmp       | .ppt  | .sql     | .vcf       | .z_     |
| .CFG        | .etl  | .mof        | .pptm | .sqlplan | .vsd       | .z01    |
| .compressed | .evt  | .mp3        | .pptx | .stp     | .wdb       | .z02    |
| .Config     | .evtx | .mpg        | .prn  | .svclog  | .wks       | .zi     |
| .cpk        | .EX   | .ms_        | .psf  | -        | .wma       | .zi_    |
| .cpp        | .ex_  | .msg        | .pst  | -        | .wmv       | .zip    |
| .cs         | .ex0  | .msi        | .pub  | -        | .wmz       | .zip_   |
| .CSV        | .FRD  | .mso        | -     | -        | .wps       | .zipp   |
| .cvr        | .gif  | .msu        | -     | -        | .wpt       | .zipped |
| -           | .guid | .nfo        | -     | -        | .wsdl      | .zippy  |
| -           | .gz   | -           | -     | -        | .wsp       | .zipx   |
| -           | -     | -           | -     | -        | .wtl       | .zit    |
| -           | -     | -           | -     | -        | -          | .zix    |
| -           | -     | -           | -     | -        | -          | .zzz    |

## <a name="close-a-support-request"></a>Cierre de una solicitud de soporte técnico

Si tiene que cerrar una solicitud de soporte técnico, [envíe un mensaje](#send-a-message) para pedir que se cierre la solicitud.

## <a name="reopen-a-closed-request"></a>Volver a abrir una solicitud cerrada

Si necesita volver a abrir una solicitud de soporte cerrada, cree un [mensaje nuevo](#send-a-message). Al hacerlo, la solicitud se volverá a abrir automáticamente.

## <a name="cancel-a-support-plan"></a>Cancelación de un plan de soporte técnico

Si tiene que cancelar un plan de soporte técnico, consulte [Cancelación de un plan de soporte técnico](../../cost-management-billing/manage/cancel-azure-subscription.md#cancel-a-support-plan).

## <a name="next-steps"></a>Pasos siguientes

[Creación de una solicitud de soporte técnico de Azure](how-to-create-azure-support-request.md)

[API REST de incidencia de soporte técnico de Azure](/rest/api/support)
