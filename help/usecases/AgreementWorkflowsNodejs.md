---
title: Flujos de trabajo del acuerdo en Node.js
description: '"[!DNL Adobe Acrobat Services] Las API incorporan fácilmente funciones de PDF en tus aplicaciones web".'
type: Tutorial
role: Developer
level: Beginner
thumbnail: KT-7473.jpg
jira: KT-7473
keywords: Destacados
exl-id: 44a03420-e963-472b-aeb8-290422c8d767
source-git-commit: 2d1151c17dfcfa67aca05411976f4ef17adf421b
workflow-type: tm+mt
source-wordcount: '2182'
ht-degree: 1%

---

# Flujos de trabajo de acuerdo en Node.js

![Banner de caso de uso Hero](assets/UseCaseAgreementHero.jpg)

Muchas aplicaciones y procesos empresariales requieren documentación, como propuestas y acuerdos. Los documentos de PDF garantizan que los archivos sean más seguros y menos modificables. También ofrecen compatibilidad con firmas digitales para que los clientes puedan completar sus documentos de forma rápida y sencilla. [!DNL Adobe Acrobat Services] Las API incorporan fácilmente funciones de PDF en tus aplicaciones web.

## Lo que puedes aprender

En este tutorial práctico, aprenda a añadir servicios de PDF a una aplicación Node.js para digitalizar un proceso de acuerdo.

## API y recursos relevantes

* [API de servicios de PDF](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/index.html)

* [API de incrustación de PDF](https://www.adobe.com/devnet-docs/dcsdk_io/viewSDK/index.html)

* [API de Adobe Sign](https://www.adobe.io/apis/documentcloud/sign.html)

* [Código del proyecto](https://github.com/adobe/pdftools-node-sdk-samples)

## Configuración de [!DNL Adobe Acrobat Services]

Para comenzar, configure las credenciales que desea utilizar [!DNL Adobe Acrobat Services]. Registre una cuenta y utilice el [Node.js Quickstart](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/index.html#node-js) para verificar sus credenciales, trabaje antes de integrar la funcionalidad en una aplicación más grande.

Primero, consiga una cuenta de desarrollador de Adobe. A continuación, en la [Introducción](https://www.adobe.io/apis/documentcloud/dcsdk/gettingstarted.html?ref=getStartedWithServicesSDK) , seleccione la *Introducción* en Crear nuevas credenciales. Puede registrarse para obtener una prueba gratuita que proporciona 1000 transacciones de documento que se pueden utilizar durante seis meses.

![Imagen de creación de nuevas credenciales](assets/AWNjs_1.png)

En la siguiente página Crear nuevas credenciales, se le pedirá que decida entre la API de incrustación de PDF y la API de servicios de PDF.

Seleccionar *API de servicios de PDF*.

Introduzca un nombre para la aplicación y marque la casilla etiquetada *Crear ejemplo de código personalizado*. Al marcar esta casilla, se incrustan automáticamente sus credenciales en el ejemplo de código. Si deja esta casilla sin marcar, deberá agregar manualmente sus credenciales a la aplicación.

Seleccionar *Node.js* para el tipo de aplicación y haga clic en *Crear credenciales*.

Momentos más tarde, se comienza a descargar un archivo .zip con un proyecto de muestra que incluye sus credenciales. El paquete Node.js para [!DNL Acrobat Services] ya se incluye como parte del código del proyecto de ejemplo.

![Imagen de selección de credenciales de API de servicios de PDF](assets/AWNjs_2.png)

## Configurar manualmente el proyecto de ejemplo

Si decide no descargar un proyecto de ejemplo desde la página Crear nuevas credenciales, también puede configurar el proyecto manualmente.

Descargue el código (sin sus credenciales incrustadas) de [GitHub](https://github.com/adobe/pdftools-node-sdk-samples). Si utiliza esta versión del código, debe añadir sus credenciales al archivo pdftools-api-credentials.json antes de utilizar:

```
{
  "client_credentials": {
    "client_id": "<client_id>",
    "client_secret": "<client_secret>"
  },
  "service_account_credentials": {
    "organization_id": "<organization_id>",
    "account_id": "<technical_account_id>",
    "private_key_file": "<private_key_file_path>"
  }
}
```

Para su propia aplicación, debe copiar el archivo de clave privada y los archivos de credenciales en el origen de la aplicación.

Debe instalar el paquete Node.js para [!DNL Acrobat Services]. Para instalar el paquete, utilice el siguiente comando:

```
npm install --save @adobe/documentservices-pdftools-node-sdk
```

## Configuración del registro

En los ejemplos se utiliza Express para el marco de trabajo de la aplicación. También utilizan log4js para el registro de aplicaciones. Con log4js, puede dirigir fácilmente el registro a la consola o a un archivo:

```
const log4js = require('log4js');
const logger = log4js.getLogger();
log4js.configure( {
    appenders: { fileAppender: { type:'file', filename: './logs/applicationlog.txt'}},
    categories: { default: {appenders: ['fileAppender'], level:'info'}}
});
 
logger.level = 'info';
logger.info('Application started')
```

El código anterior escribe los datos registrados en un archivo en ./logs/applicationlog.txt. Si desea que escriba en la consola, puede comentar la llamada a log4js.configure.

## Convertir archivos de Word en PDF

Los acuerdos y las propuestas se suelen escribir en una aplicación productiva, como Microsoft Word. Para aceptar documentos en este formato y convertirlos en PDF, puede añadir funciones para su aplicación. Veamos cómo cargar y guardar un documento en una aplicación Express y guardarlo en el sistema de archivos.

En el HTML de la aplicación, añada un elemento de archivo y un botón para iniciar la carga:

```
<input type="file" name="source" id="source" />
<button onclick="upload()" >Upload</button>
```

En el JavaScript de la página, cargue el archivo de forma asíncrona mediante la función fetch:

```
function upload()
{
  let formData = new FormData();
  var selectedFile = document.getElementById('source').files[0];
  formData.append("source", selectedFile);
  fetch('documentUpload', {method:"POST", body:formData});
}
```

Elija una carpeta para aceptar los archivos cargados. La aplicación necesita una ruta de acceso a esta carpeta. Busque la ruta absoluta mediante una ruta relativa unida a \_\_dirname:

```
const uploadFolder = path.join(__dirname, "../uploads");
```

Como el archivo se envía por correo, debe responder a un mensaje de correo en el lado del servidor:

```
router.post('/', (req, res, next) => {
  console.log('uploading')
  if(!req.files || Object.keys(req.files).length === 0) {
  return res.status(400).send('No files were uploaded');
  }
    
  const uploadPath = path.join(uploadFolder, req.files.source.name);
  var buffer = req.files.source.data;
  var result = {"success":true};
  fs.writeFile(uploadPath, buffer, 'binary', (err)=> {
    if(err) {
      result.success = false;
    }
    res.json(result);
  });       
});
```

Una vez ejecutada esta función, el archivo se guarda en la carpeta de carga de aplicaciones y está disponible para su procesamiento posterior.

A continuación, convierta el archivo de su formato nativo a PDF. El código de ejemplo descargado anteriormente contiene una secuencia de comandos denominada `create-pdf-from-docx.js` para convertir un documento en PDF. La siguiente función, `convertDocumentToPDF`, toma un documento cargado y lo convierte en un PDF de otra carpeta:

```
function convertDocumentToPDF(sourcePath, destinationPath)
{    
  try {   
    const credentials = PDFToolsSDK.Credentials
    .serviceAccountCredentialsBuilder()
    .fromFile("pdftools-api-credentials.json")
    .build();
 
    const executionContext = 
      PDFToolsSDK.ExecutionContext.create(credentials),
    createPdfOperation = PDFToolsSDK.CreatePDF.Operation.createNew();
 
    const docxReadableStream = fs.createReadStream(sourcePath);
    const input = PDFToolsSDK.FileRef.createFromStream(
      docxReadableStream, 
      PDFToolsSDK.CreatePDF.SupportedSourceFormat.docx);
    createPdfOperation.setInput(input);
 
    createPdfOperation.execute(executionContext)
    .then(result => result.saveAsFile(destinationPath))
    .catch(err => {        
      logger.erorr('Exception encountered while executing operation');        
    })
  }
  catch(err) {        
    logger.error(err);
  }
}
```

Es posible que observe un patrón general con el código:

El código genera un objeto credentials y un contexto de ejecución, inicializa alguna operación y, a continuación, ejecuta la operación con el contexto de ejecución. Puede ver este patrón a lo largo del código de ejemplo.

Al agregar algunos elementos a la función de carga para que llame a esta función, los documentos de Word que los usuarios carguen ahora se convierten automáticamente a PDF.

El código siguiente genera la ruta de acceso de destino del PDF convertido e inicia la conversión:

```
const documentFolder = path.join(__dirname, "../docs");
var extPosition = req.files.source.name.lastIndexOf('.') - 1;
if(extPosition < 0 ) {
  extPosition = req.files.source.name.length
}
const destinationName = path.join(documentFolder,  
  req.files.source.name.substring(0, extPosition) + '.pdf');
console.log(destinationName);
 
logger.info('converting to ${destinationName}')
  convertDocumentToPDF(uploadPath, destinationName);
```

## Conversión de otros tipos de archivo a PDF

El kit de herramientas de documentos convierte otros formatos en PDF, como HTML estáticos, otro tipo de documento común. El kit de herramientas acepta documentos de HTML empaquetados como archivos .zip con todos los recursos a los que hace referencia el documento (archivos CSS, imágenes y otros archivos) en el mismo archivo .zip. El propio documento de HTML debe denominarse index.html y colocarse en la raíz del archivo .zip.

Para convertir un archivo .zip que contenga HTML, utilice el código siguiente:

```
//Create an HTML to PDF operation and provide the source file to it
htmlToPDFOperation = PDFToolsSdk.CreatePDF.Operation.createNew();     
const input = PDFToolsSdk.FileRef.createFromLocalFile(sourceZipFile);
htmlToPDFOperation.setInput(input);
 
// custom function for setting options
setCustomOptions(htmlToPDFOperation);
 
// Execute the operation and Save the result to the specified location.
htmlToPDFOperation.execute(executionContext)
  .then(result => result.saveAsFile(destinationPdfFile))
  .catch(err => {
    logger.error('Exception encountered while executing operation');
});
```

La función `setCustomOptions` especifica otras configuraciones de PDF, como el tamaño de página. Aquí puede ver que la función establece el tamaño de página en 11,5 por 11 pulgadas:

```
const setCustomOptions = (htmlToPDFOperation) => {    
  const pageLayout = new PDFToolsSdk.CreatePDF.options.PageLayout();
  pageLayout.setPageSize(11.5, 8);

  const htmlToPdfOptions = 
    new PDFToolsSdk.CreatePDF.options.html.CreatePDFFromHtmlOptions.Builder()
    .includesHeaderFooter(true)
    .withPageLayout(pageLayout)
    .build();
  htmlToPDFOperation.setOptions(htmlToPdfOptions);
};
```

Al abrir un documento de HTML que contiene algunos términos, se obtiene lo siguiente en el navegador:

![Imagen de términos informáticos](assets/AWNjs_3.png)

El origen de este documento se compone de un archivo CSS y un archivo de HTML:

![Imagen de CSS y archivo de HTML](assets/AWNjs_4.png)

Después de procesar el archivo de HTML, tiene el mismo texto en formato de PDF:

![Archivo del PDF de términos del equipo](assets/AWNjs_5.png)

## Añadir páginas

Otra operación común con archivos de PDF es añadir páginas al final que puedan tener texto estándar, como una lista de términos. El kit de herramientas de documentos puede combinar varios documentos de PDF en un solo documento. Si tiene una lista de rutas de documentos (aquí en `sourceFileList`), puede agregar las referencias de archivo de cada archivo a un objeto para una operación de combinación.

Cuando se ejecuta la operación de combinación, proporciona un único archivo con información del contenido de origen. Puede utilizar `saveAsFile` en el objeto para conservar el archivo en el almacenamiento.

```
const executionContext = PDFToolsSDK.ExecutionContext.create(credentials);
var combineOperation = PDFToolsSDK.CombineFiles.Operation.createNew();
 
sourceFileList.forEach(f => {
  var combinedSource = PDFToolsSDK.FileRef.createFromLocalFile(f);
  console.log(f);
  combineOperation.addInput(combinedSource);
});
    
 
combineOperation.execute(executionContext)
  .then(result=>result.saveAsFile(destinationFile))
  .catch(err => {
    logger.error(err.message);
});    
```

## Visualización de documentos de PDF

Ha realizado varias operaciones con archivos de PDF, pero en última instancia, el usuario debe ver los documentos. Puede incrustar el documento en una página web mediante la API de incrustación de PDF de Adobe.

En la página que muestra el PDF, agregue un `<div />` para contener el documento y darle un ID. Utilizarás este identificador en breve. En la página Web, incluya un `<script />` Referencia a la biblioteca de JavaScript de Adobe:

```
<script src="https://documentcloud.adobe.com/view-sdk/main.js"></script>
```

El último bit de código que necesita es una función que muestra el documento una vez que se carga el JavaScript de la API Adobe PDF Embed. Cuando reciba una notificación de que el script se carga a través de un evento adobe_dc_view\_sdk.ready, cree un nuevo objeto AdobeDC.View. Este objeto necesita su ID de cliente y el ID del elemento creado anteriormente. Busque su ID de cliente en la [Adobe Developer Console](https://console.adobe.io/). Cuando vea la configuración de la aplicación que creó al generar las credenciales anteriormente, el ID de cliente aparecerá allí.

![Imagen de la clave de cliente de API](assets/AWNjs_6.png)

## Otras opciones de PDF

La [Demostración de la API Adobe PDF Embed](https://documentcloud.adobe.com/view-sdk-demo/index.html#/view/FULL_WINDOW/Bodea%20Brochure.pdf) permite obtener una vista previa de las otras opciones para incrustar documentos de PDF.

![Imagen de opciones del PDF de incrustación ](assets/AWNjs_7.png)

Puede activar y desactivar varias opciones y ver inmediatamente cómo se procesan. Cuando encuentre la combinación que desee, haga clic en *\&lt;/\> Generar código* para generar el código de HTML real mediante estas opciones.

![Imagen de Vista previa del código](assets/AWNjs_8.png)

## Añadir firmas digitales y seguridad

Una vez que el documento está listo, puedes añadir firmas digitales para su aprobación mediante Adobe Sign. Esta funcionalidad funciona un poco diferente de la funcionalidad que ha utilizado hasta ahora. En el caso de las firmas digitales, se debe configurar una aplicación para que utilice OAuth para la autenticación de usuarios.

El primer paso para configurar la aplicación es [registrar la aplicación](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/gstarted/create_app.md) para usar OAuth para Adobe Sign. Una vez que haya iniciado sesión, vaya a la pantalla para crear aplicaciones haciendo clic en *Cuenta*, a continuación, abra el *API de Adobe Sign* y haga clic en *Aplicaciones de API* para abrir la lista de aplicaciones registradas.

![Imagen del primer paso para registrar la aplicación](assets/AWNjs_9.png)

Para crear una nueva entrada de aplicación, haga clic en el icono más situado en la esquina superior derecha.

![Imagen del icono de signo más en la esquina superior derecha de la pantalla](assets/AWNjs_10.png)

En la ventana que se abre, introduzca un nombre de aplicación y un nombre para mostrar. Seleccionar *Cliente* del dominio y, a continuación, haga clic en *Guardar*.

![Imagen de dónde se escribe el nombre de la aplicación y el nombre para mostrar](assets/AWNjs_11.png)

Después de crear la aplicación, puede seleccionarla en la lista y hacer clic en *Configurar OAuth para la aplicación*. Seleccione las opciones. En URL de redirección, introduzca la dirección URL de la aplicación. Puede introducir varias direcciones URL aquí. Para la aplicación que está probando, el valor es:

```
http://localhost:3000/signed-in 
```

El proceso de utilizar OAuth para obtener un token es estándar. La aplicación dirige a un usuario a una dirección URL para iniciar sesión. Una vez que el usuario inicia sesión correctamente, se le redirige a la aplicación con información adicional en los parámetros de consulta de la página.

Para la dirección URL de inicio de sesión, la aplicación debe pasar el ID de cliente, la dirección URL de redirección y una lista de los ámbitos necesarios.

El patrón de la dirección URL es similar al siguiente:

```
https://secure.adobesign.com/public/oauth?
  redirect_uri=&
  response_type=code&
  client_id=&
  scope=
```

Se solicita al usuario que inicie sesión en su ID de Adobe Sign. Después de iniciar sesión, se les pregunta si desean proporcionar permisos a la aplicación.

![Imagen de la pantalla de confirmación de acceso](assets/AWNjs_12.png)

Si el usuario hace clic en *Permitir acceso* en la dirección URL de redirección, un parámetro de consulta denominado code pasa el código de autorización:

https://YourServer.com/?code=**\&lt;authorization_code>**\&amp;api_access_point=https://api.adobesign.com&amp;web_access_point=https://secure.adobesign.com

La publicación de este código en el servidor de Adobe Sign junto con su ID de cliente y secreto de cliente proporciona un token de acceso para acceder al servicio. Guarde los valores en los parámetros `api_access_point` y `web_access_point`. Estos valores se utilizan para solicitudes posteriores.

```
var requestURL = ' ${api_access_point}oauth/token?code=${code}'
  +'&client_id=${client_id}'
  +'&client_secret=${client_secret}&'
  +'redirect_uri=${redirect_url}&'
  +'grant_type=authorization_code';
request.post(requestURL, {form: { }
}, (err,response,body)=>{                
    var token_response = JSON.parse(body)
    var access_token = token_response.access_token;
    console.log(access_token);
});
```

Cuando un documento requiere una firma, primero debe cargarse. La aplicación puede cargar el documento en la `api_access_point` que se recibió al solicitar el token de OAUTH. El punto final es `/api/rest/v6/transientDocuments`. Los datos de la solicitud tienen el siguiente aspecto:

```
POST /api/rest/v6/transientDocuments HTTP/1.1
Host: api.na1.adobesign.com
Authorization: Bearer MvyABjNotARealTokenHkYyi
Content-Type: multipart/form-data
Content-Disposition: form-data; name=";File"; filename="MyPDF.pdf"
<PDF CONTENT>
```

En la aplicación, genere la solicitud con el código siguiente:

```
var uploadRequest = {
  'method': 'POST',
  'url': '${oauthParameters.signin_domain}/api/rest/v6/transientDocuments',
  'headers': {
    'Authorization': 'Bearer  ${auth_token}'
  },
  formData: {
    'File': {
      'value': fs.createReadStream(documentPath),
      'options': {
        'filename': fileName,
        'contentType': null
      }
    }
  }
};
 
request(uploadRequest, (error, response) => {
  if (error) throw new Error(error);
  var jsonResponse = JSON.parse(response.body);
  var transientDocumentId = jsonResponse.transientDocumentId;
  logger.info('transientDocumentId:', transientDocumentId)
});
```

La solicitud devuelve un `transientID` valor. El documento se ha cargado, pero aún no se ha enviado. Para enviar el documento, utilice el `transientID` para solicitar el envío del documento.

Para empezar, cree un objeto JSON que contenga la información del documento que se va a firmar. A continuación, la variable `transientDocumentId` contiene el ID del código anterior y `agreementDescription` contiene texto que describe el acuerdo que necesita firma. Las personas para firmar el documento se enumeran en `participantSetsInfo` por su dirección de correo electrónico y función.

```
var requestBody = {
  "fileInfos":[
    {"transientDocumentId":transientDocumentId}],
    "name":agreementDescription,
    "participantSetsInfo":[
      {"memberInfos":[{"email":"user@domain.com"}],
       "order":1,"role":"SIGNER"}
    ],
    "signatureType":"ESIGN","state":"IN_PROCESS"
};
```

Al enviar esta solicitud web, se genera la solicitud de firma y se devuelve un objeto JSON con un ID para la solicitud del acuerdo:

```
request(requestBody, function (error, response) {
  if (error) throw new Error(error);
  var JSONResponse = JSON.parse(response.body);
  var requestId = JSONResponse.id;
});
```

Si los firmantes se olvidan de firmar y necesitan otro correo electrónico de notificación, vuelva a enviar las notificaciones con el ID recibido anteriormente. La única diferencia es que también debe añadir los ID de participante de las partes. Puede obtener los ID de participante enviando una solicitud de GET a `/agreements/{agreementID}/members`.

Para solicitar el envío del recordatorio, primero cree un objeto JSON que describa la solicitud. El objeto mínimo necesita una lista de los ID de participante y un estado para el recordatorio (&quot;ACTIVO&quot;, &quot;COMPLETADO&quot; o &quot;CANCELADO&quot;).

La solicitud puede tener opcionalmente información adicional, como un valor para &quot;nota&quot; que se mostrará al usuario. O bien, un retraso (en horas) para esperar hasta el envío del recordatorio (en horas) `firstReminderDelay`) y una frecuencia de recordatorio (en el campo &quot;frecuencia&quot;), que acepta valores como DAILY_UNTIL_SIGNED, EVERY_THIRD_DAY_UNTIL_SIGNED o WEEKLY_UNTIL_SIGNED.

```
var requestBody = {
  //participantList is an array of participant ID strings
  "recipientParticipantIds":participantList
  ,"status":"ACTIVE",
  "note":"This is a reminder to sign out important agreement."
}
 
var reminderRequest = {
  'method': 'POST',
  'url': '${oauthParameters.signin_domain}/api/rest/v6/agreements/${agreementID}/reminders',
  'headers': {
    'Authorization': `Bearer ${access_token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(requestBody)
 
};

request(reminderRequest, function (error, response) {
});
```

Y eso es todo lo que se necesita para enviar una solicitud de recordatorio.

![Imagen de formulario web](assets/AWNjs_13.png)

## Crear formularios web

También puede utilizar la API de Adobe Sign para crear formularios web. Los formularios web permiten incrustar un formulario en una página web o vincularlo directamente. Una vez creado un formulario web, también se muestra entre los formularios web de la consola de Adobe Sign. Puede crear formularios web con el estado DRAFT para la creación incremental, el estado AUTHORING para la edición de los campos de formulario web y el estado ACTIVE para alojar inmediatamente el formulario.

![Imagen de un formulario web en la pantalla Administrar de Adobe Sign](assets/AWNjs_14.png)

Para crear un formulario web, utilice el formulario `transientDocumentId`. Elija un título para el formulario y un estado para inicializarlo.

```
var requestBody = {
  "fileInfos": [
    {
      "transientDocumentId": transientDocumentId
    }
  ],
  "name": webFormTitle,
  "state": status,
  "widgetParticipantSetInfo": {
    "memberInfos": [ { "email": "" } ],
    "role": "SIGNER"
  }
}
```

```
var createWebFormRequest = {
  'method': 'POST',
  'url': `${oauthParameters.signin_domain}/api/rest/v6/widgets`,
  'headers': {
    'Authorization': `Bearer ${access_token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(requestBody)
}
```

```
request(createWebFormRequest, function (error, response) {
  var jsonResp = JSON.parse(response.body);
  var webFormID = jsonResp.id;
});
```

Ahora puede incrustar el documento o vincularlo a él.

## Pasos siguientes

Como puede ver en los inicios rápidos y en el código proporcionado, es fácil implementar procesos de aprobación de documentos digitales y de PDF usando Node con el [!DNL Adobe Acrobat Services] API. Las API de Adobe se integran perfectamente en las aplicaciones cliente existentes.

Para descubrir los ámbitos necesarios para una llamada o ver cómo se genera, puede generar llamadas de ejemplo desde el [Documentación de la API Rest](https://secure.na4.adobesign.com/public/docs/restapi/v6). La [Inicio rápido](https://github.com/adobe/pdftools-node-sdk-samples) también se muestran otras funcionalidades y formatos de archivo [!DNL Adobe Acrobat Services] Procesos de API.

Puede añadir numerosas funciones de PDF a sus aplicaciones, lo que permite a los usuarios ver y firmar sus documentos y mucho más de forma rápida y sencilla. Para empezar, echa un vistazo [[!DNL Adobe Acrobat Services]](https://www.adobe.io/apis/documentcloud/dcsdk/) hoy.
