# DemoIsmart

----

# KashPay Guía del programador


# Change List

Version | Autor               | Fecha      | Descripcion
--------|---------------------|------------|----------------
1.0     | Emilio Betancourt   | 2018-05-01 | Version inicial
1.1     | Judá Escalera       | 2020-10-19 | UX
1.2     | Judá Escalera       | 2021-08-10 | UX
1.3     | Judá Escalera       | 2022-02-12 | UX

# Introducción

La solución KashPay posibilita realizar cobros con tarjeta bancarias. Esto se realiza a través de conectar un dispositivo lector de tarjetas, a continuación se describe cómo hacer la integración 
en la aplicación del cliente. El presente documento, tienen como finalidad mostrar cómo puede hacerse esta integración. 


# Modelo de programación

Todos los métodos provistos por el SDK KashPay se pueden considerar en una de las siguientes categorias:
1. Métodos de inicialización；
2. Métodos de interacción；
3. Métodos de notificación (listeners).

# Interface de programación

## Configuración

Procedimiento

Declarar el token de uso del lector en el archivo [gradle.properties](/gradle.properties) 

```java
    authToken=[Este valor será entregado por KashPay vía correo]
```

Agregar el repositorio donde se encuentra el componente AAR de KashPay, esto debe realizarse en el archivo
[build.gradle](/build.gradle)

```java
        maven { url "https://jitpack.io"
            credentials { username authToken }
        }
```

se deberá agregar la referencia en el archivo [build.gradle](/app/build.gradle) de la **aplicación**

```java
    implementation 'com.github.verasofty:readercore:v1.0.5'
    implementation 'com.github.verasofty:ISmartEMVLibrary:v1.0.0'
    implementation 'com.github.verasofty:ReaderIsmart:v1.0.9'
    
```

Estos serían los pasos de configuración que deben llevarse a cabo para el correcto funcionamiento del componente.


## Inicialización

Lo primero a realizar, debe ser la inicialización del lector. Para ello, se deberá ejecutar un código como el siguiente: 


```java
import com.sf.upos.reader.IHALReader;
import com.sf.upos.reader.ReaderMngr;
```

```java
    public static IHALReader reader;
    
    ...
    
    private void initReader() {
        if (reader == null) {
            Log.d(TAG, "reader ==> init---");
            reader = new HALReaderIsmartImpl();
            ((GenericReader)readerSale).setSwitchConnector( ConnectorMngr.getConnectorByID(ConnectorMngr.REST_CONNECTOR) );
        }
    }
```

## Configuración URL y usuario

Inicializar los endpoint y usuarios para la app demo, el String user_terminal deberá ser proporcionado

```java

AuthenticateData.applicationSecret = "qs4qa1ralmgb4cna";
AuthenticateData.applicationKey = "8z00pj9qxh3vaaggo7lfyw2xkj3rv80c7o1u";
AuthenticateData.applicationBundle = "test.api.service";

...
public static SharedPreferences sharedPreferences;

private void setServiceURL() {
        sharedPreferences = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());
        sharedPreferences.edit().putString(ISwitchConnector.SHARED_PREFERENCES_URL, getResources().getString(R.string.DEFAULT_URL))
                .apply();
    }

...

private final String user_terminal = "user_terminal";

```
## Leer una tarjeta

Paso 1. Configurar un objeto de la clase **TransactionDataRequest**

```java
        TransactionDataRequest request = new TransactionDataRequest();
        
        request.setUser(m_user);
        request.setLatitud(gpsLocator.getLatitud());
        request.setLongitud(gpsLocator.getLongitud());
        request.setAmount(etMonto.getText().toString());
        request.setFeeAmount(m_feeAmount);
        request.setMesero(description);
        request.setReference1(description);
        request.setReference2(description);
        request.setTransactionID(formatString(dbHelper.getNewRRC(), ZERO, 6, true));
        request.setB_purchaseAndRecurringCharge("F");
        request.setOperation(EMPTY_STRING);
```

Paso 2. Invocar la ejecución de la transacción

```java
         reader.startTransaction(this, request, 30000, this);
```

Paso 3. Esperar la notificación a través del **callback**
Nota: Se sugiere, si el contexto lo amerita, para fines de UX poner tras esta invocación un dialogo que indique al usuario que se está llevando a cabo una lecura

```java
    @Override
    public void onFinishedTransaction(final TransactionDataResult result) {        
        Log.d(TAG, "== onFinishedTransaction() ==");
        
        if (result.getResponseCode() == 0) {  
            processSuccessTransaction(result);
        } else {
            processError(result);
        }
    }
```
