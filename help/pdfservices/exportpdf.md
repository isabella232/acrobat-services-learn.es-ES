---
title: Uso de la API de servicios de PDF para exportar PDF a Word, PowerPoint y mucho más
description: Aprenda a ejecutar la operación de exportación de la API de servicios de PDF mediante archivos de ejemplo para los lenguajes Node.js, Java y .Net
type: Tutorial
role: Developer
level: Intermediate
thumbnail: KT-6674.jpg
kt: 6674
exl-id: 55f5b04e-0249-47d9-9131-2f9ec01db7e8
source-git-commit: 2d1151c17dfcfa67aca05411976f4ef17adf421b
workflow-type: tm+mt
source-wordcount: '500'
ht-degree: 5%

---

# Uso de la API de servicios de PDF para exportar PDF a Word, PowerPoint y mucho más

![Crear imagen de héroe de PDF](assets/ExportPDF_hero.jpg)

La API de servicios de Adobe PDF convierte archivos de PDF en MS Office, texto e imágenes mediante API. Hay muchos casos prácticos habituales para desbloquear PDF existentes para la edición y el análisis de contenido, y con PDF Services los desarrolladores de API pueden integrar fácilmente esta capacidad en los sistemas y aplicaciones existentes. Convierte archivos de PDF a MS Word para editar contenido, aprobaciones y enviarlos posteriormente para su firma con el fin de crear flujos de trabajo de contratos personalizados. O bien, exporta el contenido del PDF al formato de MS Excel para cálculos de facturas y financieros o análisis de datos.

La operación de exportación admite las siguientes conversiones de archivos de PDF:

* PDF a Microsoft Word (DOC, DOCX)
* PDF a Microsoft PowerPoint (PPTX)
* PDF a Microsoft Excel (XLSX)
* PDF a texto (RTF)
* PDF a imagen (JPEG, PNG)

En este tutorial, aprenderá los conceptos básicos sobre cómo ejecutar la primera operación de exportación de la API de Servicios de PDF mediante archivos de ejemplo para los idiomas Node.js, Java y .Net.

## Paso 1: Cree sus credenciales y configure su entorno:

Utilice los siguientes tutoriales de introducción para crear credenciales de API, descargar archivos de muestra y configurar su entorno.

[Introducción a la API de servicios de PDF y Java](gettingstartedjava.md)

[Introducción a la API de servicios de PDF y .Net](gettingstartednet.md)

[Introducción a la API de servicios de PDF y Node.js](createpdffromhtml.md)

## Paso 2: Ejecute la operación de exportación a PDF con los archivos de muestra

**Java**

1. Abra un símbolo del sistema.

1. Cambie los directorios en el directorio de código de ejemplo.

   Por ejemplo, C:\Temp\PDFToolsAPI\adobe-dc-pdf-tools-sdk-java-samples

1. Ejecute el siguiente comando::

   `mvn -f pom.xml exec:java -Dexec.mainClass=com.adobe.platform.operation.samples.exportpdf.ExportPDFToDOCX`

El PDF se creará en el directorio src/main/resources.

**.Net**

1. Abra un símbolo del sistema.

1. Cambie los directorios en el directorio de código de ejemplo.

   Por ejemplo, C:\Temp\PDFToolsAPI\adobe-dc-pdf-tools-sdk-NetSamples

1. Vuelva a cambiar los directorios en el directorio ExportPDFtoDocx .

1. Ejecute el siguiente comando::

   `dotnet run ExportPDFToDocx.csproj`

El PDF se creará en el mismo directorio.

**Node.js**

1. Abra un símbolo del sistema.

1. Cambie los directorios en el directorio de código de ejemplo.

   Por ejemplo, C:\Temp\PDFToolsAPI\adobe-dc-pdf-tools-sdk-node-samples

1. Ejecute el siguiente comando::

   `node src/ocr/ocr-pdf.js`

El PDF se crea en la ubicación designada en la salida, que de forma predeterminada es el directorio pdfServicesSdkResult.

## Pensamientos finales

Ahora debería tener un ejemplo válido que se pueda importar a las aplicaciones existentes para iniciar una prueba de concepto. En cada uno de los directorios de ejemplo, puede ver otro ejemplo para exportar archivos de PDF al formato de imagen. Los mismos pasos anteriores le permiten ejecutar también ese ejemplo. Para cambiar a otro formato, puede actualizar el código al nuevo formato que desee:

SupportedTargetFormats.PPTX

Y el resultado del destino:

output/exportPdfOutput.PPTX

A otro formato.

## Recursos y pasos siguientes

* Para obtener más ayuda y asistencia, visite la [[!DNL Adobe Acrobat Services] API](https://community.adobe.com/t5/document-cloud-sdk/bd-p/Document-Cloud-SDK?page=1&amp;sort=latest_replies&amp;filter=all) foro de la comunidad

* API de servicios de PDF [Documentación](https://www.adobe.com/go/pdftoolsapi_doc)

* [FAQ](https://community.adobe.com/t5/document-cloud-sdk/faq-for-document-services-pdf-tools-api/m-p/10726197) para preguntas de API de servicios de PDF

* [Contacte con nosotros](https://www.adobe.com/go/pdftoolsapi_requestform) para preguntas sobre licencias y precios
