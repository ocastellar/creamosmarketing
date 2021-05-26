# creamosmarketing
Documentacion del API de ilio, para integraciones con plataformas de terceros, con el API es posible crear y consultar polizas. Los detalles sobre como autenticarse y utilizar el API los pueden encontrar en Trabajando con el API.

### Trabajando con el API

El API contiene toda la funcionalidad que necesitas para crear y consultar una poliza.  
  1. Crear una poliza
  2. Consultar poliza
  3. Recibir callback con la poliza 

Una vez creada la poliza el asegurado recibira una copia en su correo y el asesor recibe la url con la poliza para descargar en PDF.
El API de ilio se puede consumir por medio de request HTTPS realizadas a las urls:

Producción: https://ilio.creamosmarketing.com/

Pruebas: https://stage.creamosmarketing.com/

La documentación está escrita usando la URL de Pruebas, cuando quieras usar el servicio en Producción solo cambiala a la url correspondiente.
Lo primero que necesitas para usar el api es saber como autenticarte.


### Autentificación

Para utilizar el API necesitaras 2 cosas, la API key y un token de acceso. La API key es necesaria para acceder a la api de ilio de forma general. Mientras que el token de acceso es necesario para realizar operaciones usando tu cuenta.

Estas credenciales se deben incluir en los headers de tus request al api. Authorization y x-api-key.

por ejemplo, para consultar la poliza usando Javascript harias algo asi:

 

 ###### Un simple GET con el header de Authorization y su respectivo Token

``` 
let response = await fetch("https://stage.creamosmarketing.com/polizas", {
        method: 'GET',
        headers: {
            Accept: 'application/json',
            'Content-Type': 'application/json',
            'Authorization': "Bearer TU_ACCESS_TOKEN",
            'x-api-key' : "TU_LLAVE_PUBLICA"
        },
    });
```

Ten en cuenta que en el header de Authorization hay un espacio entre la palabra Bearer y el token de acceso.


#### ¿Como accedo a la API de Ilio?

Una vez tengas un usuario en la plataforma ilio se te generara el x-api-key y el refresh token.
Una vez tengas tu refresh token, accedes al siguiente endpoint:

```
URL: https://stage.creamosmarketing.com/token/refresh/
METHOD: POST
Content-Type: application/json
```
El cuerpo de tu request debe ser un objeto de json convertido a texto (text/json), las propiedades que se deben enviar son:

| Parametro | Tipo | Descripción |
| -- | -- | -- |
| refresh | String | 	El refresh token |
 


La respuesta que deberias obtener tendra estos parametros:

| Parametro | Tipo | Descripción |
| -- | -- | -- |
| access | String | Token de acceso con una validez de 24 horas |
 
Por ejemplo para obtener tu token de acceso usando javascript:

```
let response = await fetch("https://stage.creamosmarketing.com/token/refresh/", {
        method: 'POST',
        headers: {
            Accept: 'application/json',
            'Content-Type': 'application/json',
            'x-api-key' : "asdfghjkl1234456"
        } ,
        body: JSON.stringify({
            refresh: "TU_REFRESH_TOKEN",
    }),
});

if (response.status == 200){
    console.log(await response.json());
}

//la respuesta en consola seria:

{
    "access": "TU_NUEVO_ACCESS_TOKEN"
}
```
El token recibido va a servirte para realizar peticiones por otras 24 horas, en ese momento deberás renovarlo, usando este mismo endpoint.

####  ¿Que viene ahora?

Veamos como puedes crear tu primera solicitud y obtener la poliza.

### Crear una poliza

El endpoint para crear es:

``` 
URL: https://stage.creamosmarketing.com/solicitud/
Method: POST
Content-Type: application/json
```
El cuerpo de tu request debe ser un objeto de json convertido a texto, las propiedades que se deben enviar y su contenido son:
 
| Parametro | Tipo | Descripción |
| -- | -- | -- |
| id_operador_comercial | int | el id del operador comercial dentro de ilio |
| id_asesor_comercial | int | id del asesor comercial asignado al operador |
| id_entidad_recaudo | int | entidad recaudadora |
| numero_contrato | int | numero del contrato valido|
| fecha_venta | date | formato DD/MM/AAAA|
| id_aseguradora | int | id de la entidad con la que se toma el seguro ( 7 = Liberty Seguros S.A., 11 = Seguros Alfa S.A. )|
| id_producto | int | id del producto relacionado con la aseguradora|
| id_plan | int | id del plan relacionado al producto|
| asegurados | Array | listado con los datos del tomador ò los tomadores de la poliza con sus beneficiarios|
 
 

#####  Posibles respuestas
| Código HTTP | Causa | 
| -- | -- |
| 201 | Solicitud creada | 
| 400 | Faltan campos o algun campo invalido |  

Si todo es correcto recibiras una respuesta con estado 201(creado) y un objeto json con estos parametros:

| Parametro | Tipo | Descripción |
| -- | -- | -- |
| num_poliza | int | id de la poliza generada por ilio |
| estado_poliza | int | estado de la solicitud 10: DIGITADA WEB. 33: EN PROCESO DE AUDITORIA. 34: SOLICITUD RECHAZADA. 3: pagado. 104: ERROR DE CREACION ......
| mensaje | String | descripcion del estado de la poliza |
| url_poliza | String | Código alfanumerico de identificacion de la poliza. Con este se accede a la vista web en https://stage.creamosmarketing.com/solicitud/{url_poliza} |
| platform | String | plataforma por donde se tramito el seguro |

 

Por ejemplo para crear una solicitud usando Javascript y procesar la respuesta:

``` 
 
let newSolicitud =[{
        id_operador_comercial: 54,  
        id_asesor_comercial: 1552, 
        id_entidad_recaudo: 9,  
        numero_contrato: 1133773,  
        fecha_venta : "25/05/2021",  
        id_aseguradora: 11,  
        id_producto: 18,  
        id_plan: 484,  
        asegurados:[{
             asegurado:{
                  tipoDoc:"CC",
                  documento:"1129532875",
                  nombre1:"OSVALDO",
                  nombre2:"ANDRES",
                  apellido1:"CASTELLAR",
                  apellido2:"AHUMADA",
                  nombre:"OSVALDO ANDRES CASTELLAR AHUMADA",
                  telefono:"3054439847",
                  celular:"3054439847",
                  email:"ing.ocastellar@gmail.com",
                  sexo:"M",
                  "fechaNacimiento":"07/12/1986",
                  "fechaExpedicion":"31/01/2005"                 
               },
               beneficiarios:[{porcentaje:50.0, parentesco_id: 13, nombre:"SEIGRIS CONTRERAS"}, 
                              {porcentaje:50.0, parentesco_id: 3, nombre:"VALERIA CASTELLAR"}]
         }]
 }]
    let response = await fetch("https://stage.creamosmarketing.com/solicitud/", {
            method: 'POST',
            headers: {
                Accept: 'application/json',
                'Authorization': "Bearer TU_ACCESS_TOKEN",
                'Content-Type': 'application/json',
                'x-api-key' : "asdfghjkl1234456"
            },
            body: JSON.stringify(newSolicitud),
    });

    if(response.status.toString() == "201"){ 
            let res = await response.json();
            console.log(res);
    }

    //el output de la consola seria:
    {
        num_poliza : 2272889,
        estado_poliza: 10, // estado interno deacuerdo al modelo de negocio
        mensaje: "DIGITADA WEB", 
        url_poliza: "ct9yd3g3",
        platform: "ASESOR DIGITAL GASES DEL CARIBE "
    }
```

### Consultar poliza

Para obtener los detalles de una solicitud se usa este endpoint:

``` 
URL: https://stage.creamosmarketing.com/solicitud_detail/{num_poliza}
Method: GET
Content-Type: application/json
```

{num_poliza} es la url alfa númerica que recibes al crear la poliza, solo puedes consultar la poliza que te pertenescan, y tu request debe estar autenticada con el header de Authorization, como se menciona en el Apartado de Autentificación.


este endpoint devuelve un objeto de json con estos parametros en su body:


| Parametro | Tipo | Descripción |
| -- | -- | -- |
| num_poliza | int | id de la poliza generada por ilio |
