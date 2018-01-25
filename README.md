**Anmerkung**: Diese Dokumentation verwendet das generische Maskulin (z. B. Nutzer) um die Lesbarkeit des Textes zu erhöhen. Es impliziert gleichermaßen die weibliche Form (Nutzerin).

Wanderlust ist eine App für Backpacker. Sie soll ihnen dabei helfen interessante Orte zu finden, indem sie ihre Lieblingsorte mit anderen Backpackern teilen. Dabei steht der persönliche Kontakt im Vordergrund. Backpacker teilen ihre Lieblingsorte nur mit Personen, die sie persönlich getroffen haben. Das erhöht die Relevanz der Empfehlungen und schränkt die Reizüberflutung an, denen Backpackern in einer neuen Stadt sowieso ausgesetzt sind.

An Wanderlust sind drei Komponenten beteiligt. Es gibt die eigentliche App, welche vom Nutzer verwendet wird, eine Administrationsoberfläche, sowie ein Backend, in dem die gesamten Daten verwaltet werden und das über eine Schnittstelle von den anderen beiden Komponenten ansprechbar ist. Jede dieser Komponente befindet sich in einem eigenen Repository. Im Folgenden sollen diese Komponenten näher beschrieben werden.

# Android-Anwendung

Die Android-Anwendung ist die Komponente, die tatsächlich direkt vom Nutzer verwendet wird. Eine ensprechende Webanwendung ist derzeit nicht vorgesehen, ließe sich aber aufgrund der losen Kopplung zwischen REST API und Android-Anwendung relativ problemlos dem Gesamprodukt hinzufügen.

Mithilfe der Android-Anwendung können Nutzer Orte über Google Maps auswählen, diese mit einem Kommentar, Bildern sowie verschiednen Tags versehen und abspeichern. Gespeicherte Orte können dann als Favoriten markiert werden, um zu signalisieren, dass diese Orte mit Freunden geteilt werden sollen. Gespeicherte Orte lassen sich zum einen in Form einer Liste darstellen, zum anderen können sie aber auch in einer Kartenansicht eingesehen werden.

Um seine Favoriten-Liste mit anderen Backpackern zu teilen kann der Nutzer diese als Freund hinzufügen. Das kann über NFC oder über eine E-Mail-Adresse geschehen. Sobald einem Zugriff auf die Orte eines anderen Backpackers gewährt wurde, werden diese gemeinsam mit den Orten anderer Freunde auf einer Karte angezeigt. Dabei lassen sich über einen Filter die Orte bestimmter Freunde ausblenden. Klickt man auf einen Marker auf der Karte, erhält man die Kommentare und Bilder aller Freunde, die diesen Ort in ihrer Favoritenliste haben.

Für jeden gespeicherten Freund gibt es eine Detailansicht, die Daten des Nutzers und seine favorisierten Orte anzeigt. Zudem wird eine Möglichkeit geboten, die Orte nach zugehörigem Staat zu filtern. 

## Projektstruktur

Im Folgenden soll der Aufbau der Android-Anwendung näher beschrieben werden. Dabei wird auf die Ordner- und Packagestruktur eingegangen, sowie die Zuständigkeit der verschiedenen Klassen beleuchtet. Sehr bedeutend ist hierfür auch eine kurze Beschreibung der Funktionalität der einzelnen UI-Komponenten wie Activities und Fragments.

Die von Android Studio angelegte Verzeichnisstruktur wird in der entsprechenden [Dokumentation](https://developer.android.com/studio/projects/index.html) erläutert. Im Rahmen dieser Dokumentation ist das Verzeichnis */app/src/main* von Bedeutung. Java-Dateien befinden sich dabei unter */app/src/main/java*, Ressourcen wie Bilddateien oder XML-Spezifikationen befinden sich im Verzeichnis */app/src/main/res*.

```
├───app
│   ├───keystore
│   └───src
│       ├───androidTest
│       │   └───java
│       │       └───com
│       │           └───interactivemedia
│       │               └───backpacker
│       ├───main
│       │   ├───java
│       │   │   └───com
│       │   │       └───interactivemedia
│       │   │           └───backpacker
│       │   │               ├───activities
│       │   │               ├───adapters
│       │   │               ├───fragments
│       │   │               ├───helpers
│       │   │               ├───models
│       │   │               └───services
│       │   └───res
│       │       ├───drawable
│       │       ├───drawable-v24
│       │       ├───layout
│       │       ├───menu
│       │       ├───mipmap-anydpi-v26
│       │       ├───mipmap-hdpi
│       │       ├───mipmap-mdpi
│       │       ├───mipmap-xhdpi
│       │       ├───mipmap-xxhdpi
│       │       ├───mipmap-xxxhdpi
│       │       ├───values
│       │       ├───values-w820dp
│       │       └───xml
│       └───test
│           └───java
│               └───com
│                   └───interactivemedia
│                       └───backpacker
│                           └───fragments
└───gradle
    └───wrapper
```



Die Packages im Verzeichnis */app/src/main/java* spalten den Quellcode in verschiedene Aufgabenbereiche. Jedes Package ist für einen anderen Teil der Anwendungslogik zuständig. Eine detaillierte Beschreibung der Packages erfolgt in den JavaDocs.

Im Package ***activities*** befinden sich sämtliche Activities, sprich die komplette UI-Logik. Es implementiert die gesamte Interaktion zwischen Nutzer und App. Da keine strikte Trennung von View- und Controller-Komponenten besteht, enthalten Activities auch Business-Logik, wie beispielsweise die Interaktion mit dem Backend. Die Darstellung des UI wird durch eine entsprechende XML-Datei in */app/src/main/res* definiert.

***fragments*** implementiert Klassen, die von den Activities verwendet werden. Sie sind an keine bestimmte Activity gebunden, sondern lassen sich einfach wiederverwenden. Wichtige Fragmente sind *MyMapFragment*, *MyListFragment*, *MyFriendsFragment* und *SettingsFragment*. Sie sind in die *HomeActivity*, quasi der Hauptanlaufstelle der App, eingebunden. Der Nutzer kann dort zwischen diesen Fragmenten navigieren.

Das Package ***helpers*** stellt Hilfsklassen zur Verfügung, welche die Implementierung der Logik vereinfachen. Sie reduzieren die Komplexität der Activities und Fragmente, reduizieren Redundanz und verbessern die Lesbarkeit. Die Hilfsfunktionen sind statisch implementiert, es wird also keine Instanziierung der Hilfsklassen benötigt. Da es sich um reine Hilfsfunktionen handelt, muss der Kontext wie beispielsweise Shared Preferences als Parameter übergeben werden.

Das Package ***adapters*** enthält von Adapterklassen wie dem *ArrayAdapter* abgeleitete Klassen, mit denen die *ListViews* innerhalb der Activities oder Fragments befüllt und Änderungen behandelt werden.

Im ***services***-Package befinden sich Service-Klassen, die für Push-Benachrichtigungen benötigt werden.

Im Package ***models*** befinden sich verschiedene Modelklassen, die im folgenden Abschnitt beschrieben werden.

## Models

Die im Package *models* definierten Modellklassen repräsentieren Objekte der realen Welt. Sie besitzen zwar Gegenstücke im Backend, sind aber tatsächlich unabhängig voneinander. Sie können durchaus Unterschiede aufweisen. Die clientseitigen Models werden auch von Gson genutzt, um die JSON-formatierten Server-Antworten in Java-Objekte umzuwandeln. Modellklassen stellen neben den Daten auch entsprechende Getter-Methoden zur Verfügung um auf diese zuzugreifen.

### User

Das *User* Model repräsentiert Backpacker, die Wanderlist verwenden. Es wird sowohl für den eingeloggten Nutzer, als auch für seine Freunde genutzt. Ein User besitzt neben seinen persönlichen Informationen auch eine Freundesliste und eine Liste hinzugefügter Orte. Außerdem kann er ein Profilbild besitzen, das über eine URL definiert wird.

### Location

Das *Location* Model repräsentiert tatsächliche Orte, mitsamt der Eigenschaften, die ein User ihnen zuschreibt. Ein Ort kann also mehr als einmal existieren, solange mehrere User ihn angelegt haben. Informationen wie Beschreibungen und Kategorien können sich dabei unterscheiden. Eine Location ist immer einem bestimmten Nutzer zugeordnet, ein User kann aber mehrere Locations besitzen. Da eine Location auch eine googleId besitzen kann, können Locations, die den selben physikalischen Ort beschreiben, erkannt werden. So wird beispielsweise im MapFragment jeder Ort nur einmal eingezeigt, unabhängig davon, wie viele Nutzer ihn in ihrer Favoritenliste haben.

Einige Eigenschaften wie der Name, die Google-ID, die Stadt, das Land und die Koordinaten, werden durch die App vorgenommen. Die entsprechenden Informationen stellt der *PlacePicker* bereit. Andere Eigenschaften wie die passenden Kategorien, einer Beschreibung und Bildern können vom Benutzer festgelegt werden.

## Manifest

In der Datei *AndroidManifest.xml* werden die benötigten Berechtigungen definiert. Diese muss der Nutzer bestätigen, um die App aus dem Play Store herunterzuladen. Dies ist notwendig, damit die App auf bestimmte Sensoren oder Funktionen des Handys zugreifen kann..

Berechtigungen werden in der Form `<uses-permission android:name="android.permission.INTERNET" />` angegeben. Folgende Berechtigungen benötigt Wanderlust:

| Berechtigung           | Für welche Funktion?                     |
| ---------------------- | ---------------------------------------- |
| INTERNET               | Kommunikation über das Internet mit dem Backend und den Google APIs. |
| ACCESS_NETWORK_STATE   | Einholen von Netzwerkinformationen, um z.B. herauszufinden ob eine Internetverbindung besteht. |
| ACCESS_FINE_LOCATION   | Zugriff auf die genaue Position des Handys: Wichtig für den *PlacePicker*. |
| WRITE_EXTERNAL_STORAGE | Schreibzugriff auf den externen Speicher: Zum Speichern der aufgenommenen Bilder in den Speicher, um sie daraufhin hochladen zu können. |
| READ_EXTERNAL_STORAGE  | Lesezugriff auf den externen Speicher: Zum Abrufen der aufgenommenen Bilder oder zum Auswählen von bereits gespeicherten Bildern für das eigene Profil oder eine neue Location. |
| NFC                    | Zugriff auf NFC Funktionen: Zum Hinzufügen neuer Freunde. |

Ab Android 6.0 müssen für als gefährlich eingestufte Berechtigungen zusätzlich eine Zustimmung zur Laufzeit eingeholt werden. Siehe hierzu diesen [Leitfaden](https://developer.android.com/training/permissions/requesting.html). Als gefährlich wird dabei auch der Zugriff auf den externen Speicher eingestuft.

Zudem wird im Manifest angegeben, welche Features die App verwendet. Dies wird in der Form `<uses-feature android:name="android.hardware.camera" />` eingetragen. Wanderlust verwendet NFC um Freunde hinzuzufügen und die Kamera um Profilbilder und Location-Bilder hochzuladen.

## Bilder-Verwaltung

Für das asynchrone Herunterladen und Anzeigen der auf dem Server gespeicherten Bilder wird die Bibliothek *[Glide](https://bumptech.github.io/glide/)* in der 4. Version verwendet. Diese erleichtert es den Entwicklern immens, den Code für das Herunterladen von Bildern (ohne die umständlichere Nutzung von *HttpUrlConnection*) auf einfache Weise zu implementieren. Der Code wird dabei sehr schlank gehalten. Außerdem bietet die Bibliothek Möglichkeiten, Callbacks zu implementieren, die ausgeführt werden, sobald ein Bild heruntergeladen wurde. Ebenso ist in Glide bereits eine Caching-Strategie integriert, die dazu führt, das bereits heruntergeladene Bilder nicht nochmals geladen werden müssen. Die Api, um Glide zu benutzen, sieht für einen einfachen Beispielsfall wiefolgt aus:

```java
Glide.with(context).load(url).into(imageView);
```

Über die Klasse *RequestOptions* ist es möglich, bestimmte Optionen für das Laden und/oder Anzeigen der Bilder zu definieren. Im folgenden Beispiel wird festgelegt, dass das Bild bei jeder Ausführung des Codes neu vom Server heruntergeladen werden soll. Dies ist in der *EditProfileActivity* nötig, damit, nachdem das Bild vorher vom Nutzer geändert wurde, das aktuelle Bild beim erneuten Aufrufen der Activity angezeigt wird. 

```java
RequestOptions requestOptions = new RequestOptions()
  	.diskCacheStrategy(DiskCacheStrategy.NONE)
  	.skipMemoryCache(true);
Glide.with(context).load(url).apply(requestOptions).into(imageView);
```

## Serialisierung

Zur Serialisierung  und Deserialisierung von Java-Objekten zu bzw. von JSON verwenden wir die Bibliothek [*Gson*](https://github.com/google/gson) von Google. Diese macht als den Entwicklern deutlich einfacher als die Verwendung der *JSONObject* Klasse, mit in JSON formatiertem Text umzugehen. Durch Gson kann die Antwort des Servers in die bereits beschriebenen Model-Klassen User und Location umgewandelt werden. Auch können Objekte dieser Klassen in einen String in JSON transformiert werden, um diesen als Body im Request an den Server mitzuschicken. Des Weiteren können individuelle Strategien angelegt werden, um z.B. bestimmte Felder zu ignorieren oder auch umzubennen. 

In diesem Beispiel wird aus einem JSON String ein neues Object der User Klasse erstellt, auf dessen Funktionen daraufhin wie gewohnt zugegriffen werden können.

```java
Gson gson = new Gson();
User user = gson.fromJson(result, User.class);
String firstName = user.getFirstName();
```

Das nächste Beispiel ist entnommen aus der *AddLocationActivity*, in der aus einem Objekt der Klasse Location ein String entsteht, der als Request Body an den Server geschickt wird. 

```java
Location location = new Location(
                        place.getId(),
                        userId,
                        place.getName().toString(),
                        true,
                        description,
                        selectedCategories,
                        new double[]{place.getLatLng().latitude, place.getLatLng().longitude},
                        city,
                        country
                );
                //transform to json via gson
                Gson gson = new Gson();
                String locationJson = gson.toJson(location);
```

## Push Notifications

Für die Integration von Push Notifications vom Server zum Android Client wird die *[Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/ )* Bibliothek von *Firebase* (mittlerweile Teil von Google) verwendet. FCM hat *Google Cloud Messaging* als die von Google empfohlene Lösung zur Synchronisation von Server und Client abgelöst. Für Details zur Verwendung und Implementierung lesen sie bitte das Kapitel Push Notifications.

## PlacePicker

Der [*PlacePicker*](https://developers.google.com/places/android-api/placepicker) ist ein UI-Widget, das auf die Google Places API zugreift und mithilfe dessen man es dem Nutzer ermöglicht, auf einer Karte oder über ein Suchfeld einen Ort auszuwählen. 

Der PlacePicker wird in die *AddLocationActivity* integriert. Über den Aufruf von *startActivityForResult* wird der PlacePicker geöffnet. Sobald der Nutzer einen Ort ausgewählt hat, wird in der Activity die Funktion *onActivityResult* aufgerufen, in der auf die Daten dieses Ortes zugegriffen werden kann. 

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == PLACE_PICKER_REQUEST) {
            if (resultCode == RESULT_OK) {
                Place place = PlacePicker.getPlace(this, data);
               	String name = place.getName();
                ...
             }
        }
}
```



<img src="https://developers.google.com/places/images/placepicker.png" alt="PlacePicker" style="width: 200px;"/>

​					https://developers.google.com/places/images/placepicker.png



## Google Maps API

Die Einbindung der [*Google Maps API*](https://developers.google.com/maps/documentation/android-api/?hl=de) für Android wird in das *MyMapFragment*, in dem dem Nutzer die Kartenansicht mit seinen gespeicherten Orten und den Favoriten seiner Freund angezeigt werden, integriert. Es gibt zwei Möglichkeiten, die Karte einzubinden: das *MyMapFragment* (nicht zu verwechseln mit dem des Projekts) und die *MapView*. Dadurch, dass die *BottomNavigation* in der *HomeActivity* mit verschiedenen Fragments arbeitet und die Verschachtelung mehrere Fragments nicht empfohlen wird, greifen wir hier zur zweiten Variante. Hierfür wird die *MapView* in die zum Fragment zugehörige xml-Datei eingefügt

```xml
<com.google.android.gms.maps.MapView
	android:id="@+id/map_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

 In unserem *MyMapFragment* kann nun auf die Karte zugegriffen werden.

```java
mapView = view.findViewById(R.id.map_view);
mapView.onCreate(mapViewBundle);

mapView.getMapAsync(new OnMapReadyCallback() {
	@Override
    public void onMapReady(GoogleMap googleMap) {
    	map = googleMap; //map is a global variable of type GoogleMap
		loadLocationsOfUser();
	}
});
```

Die Methode *loadLocationsOfUser* schickt einen Request an unsere API, um die gespeicherten Orte des Nutzers und daraufhin auch die Favoriten seiner Freunde zu erhalten. Für jede Location eines jeden Users wird nun ein [*Marker*](https://developers.google.com/maps/documentation/android-api/marker?hl=de) auf der Karte gesetzt. Dabei greifen wir auf Methoden der Location Klasse zu, um die Koordinaten des Markers, sowie den Titel zu setzen. Je nachdem, welchem Freund der Ort angehört, wird dementsprechend eine andere Farbe des Markers ausgewählt. 

```java
//computeColor uses the index of the list of users to select the marker's color
BitmapDescriptor icon = BitmapDescriptorFactory.defaultMarker(MarkerColors.computeColor(index));

MarkerOptions options = new MarkerOptions()
		.position(new LatLng(location.getCoordinates()[0], location.getCoordinates()[1]))
         .title(location.getName())
         .icon(icon);

Marker marker = map.addMarker(options);
```

Zum einfacheren Verständnis der Funktionsweise des Codes, wurden in diesem Codeschnipsel zusätzliche Logik wie das Setzen eines anderen Icons, wenn der gleiche Ort von zwei unterschiedlichen Nutzer gespeichert wurde, ausgelassen. 

Nicht zu vergessen ist die Angabe des in der Google Console kreierten API Keys in der Datei *AndroidManifest.xml*.

```xml
<meta-data
	android:name="com.google.android.geo.API_KEY"
    android:value="AIzaSyA9yABz8sHgpRXtGuwzkgbEMY4HbqLpUwg" />
```

## Google Sign In

Für die Authentifizierung des Nutzers verwenden wir Google OAuth. Die dafür notwendige Logik befindet sich in der *LoginActivity*. In der *onCreate* Methode wird Google Sign In konfiguriert:

```java
// Configure sign-in to request the user's ID, email address, the ID token, and basic
// profile. ID and basic profile are included in DEFAULT_SIGN_IN.
GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
	.requestEmail()
    .requestIdToken(getString(R.string.server_client_id))
    .build();

// Build a GoogleSignInClient with the options specified by gso.
mGoogleSignInClient = GoogleSignIn.getClient(this, gso);
```

Zu beachten ist hierbei, dass wir explizit angeben müssen, welche Daten wir benötigen. Um den *IdToken*, den wir benötigen werden, um uns im Backend zu authentifizieren, müssen wir dafür die in der Google API Console hinterlegte *client ID* als Argument übergeben. Dieser Token wird daraufhin in den *SharedPreferences* gespeichert und bei jedem zukünftigen Request an den Server im Header mitgegeben, sodass im Backend überprüft werden kann, um welchen Nutzer es sich handelt und ob dieser für diese Anfrage berechtigt ist. 

In der *onStart* Methode der *LoginActivity* wird ein sogenannter *silentSignIn* ausgeführt, um somit zu überprüfen, ob der Nutzer bereits eingeloggt ist. Ist dies nicht der Fall, so wird dem Nutzer das User Interface der Activity angezeigt. Sobald der Nutzer den Login Button klickt, wird folgende Methode aufgerufen:

```java
private void signIn() {
	Intent intent = mGoogleSignInClient.getSignInIntent();
	startActivityForResult(intent, SIGN_IN_REQUEST);
}
```

Das Resultat des Logins wird daraufhin in der *onActivityResult* Methode behandelt.

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	super.onActivityResult(requestCode, resultCode, data);

	// Result returned from launching the Intent from GoogleSignInClient.getSignInIntent(...);
    if (requestCode == SIGN_IN_REQUEST) {
	// The Task returned from this call is always completed, no need to attach a listener.
	Task<GoogleSignInAccount> task = GoogleSignIn.getSignedInAccountFromIntent(data);
	handleSignInResult(task);
    }
}
```

In der aufgerufenen Methode *handleSignInResult* wird daraufhin der ID Token persistent auf dem Gerät gespeichert und ein Request an den Server geschickt, um den Nutzer, falls er noch nicht existiert, zu erstellen. 

## NFC

Die Applikation soll den Nutzern eine möglichst interessante und einfache Möglichkeit bieten ihre Orte miteinander zu teilen, sprich eine Freundesbeziehung aufzubauen. Hier haben wir uns dafür entschieden auf einen drahtlose Übertragung von Daten zu setzen, da den Nutzern somit eine lästige Suche von Nutzern durch Texteingabe erspart bleibt. Da es für uns nur nötig ist, eine Id der Nutzer auszutauschen, reicht für diese Übertragung NFC vollkommen aus. Außerdem bietet NFC die intuitive Möglichkeit, die Freundesbeziehung aufzubauen, indem beide Handys aneinandergelegt werden. Zur Nutzung von NFC werden die nötigen Berechtigungen im Android Manifest festgelegt.

```xml
<uses-permission android:name="android.permission.NFC" />
<uses-feature android:name="android.hardware.nfc" />
```

Hierbei ist es nicht nötig, anzugeben, dass NFC benötigt wird (`<uses-feature ... required="true" />` ), da die Applikation nicht darauf angewiesen ist, dass das Gerät NFC besitzt. Ist dies zum Beispiel der Fall, wird zum "Fallback" über die Suche eines anderen Nutzers über seine E-Mail-Adresse weitergeleitet. Also soll die App auch Nutzern im Play Store angezeigt werden, die kein NFC besitzen.

Die Logik zum Aufbau der NFC Verbindung ist in der *AddFriendNfcActivity* implementiert. In der *onCreate* Methode wird auf die Klasse *NfcAdapter* zugegriffen und die jeweiligen Callbacks bei Erkennen eines anderen Gerätes und beim erfolgreichen Senden einer Nachrichten festgelegt.

```java
NfcAdapter nfcAdapter = NfcAdapter.getDefaultAdapter(this);
//check if nfc is available
if (nfcAdapter == null) {  
	//if that is the case we redirect to the add friend via email activity
    ...
} else {
	//check if NFC is enabled
    if (nfcAdapter.isEnabled()) {
    	//This will refer back to createNdefMessage for what it will send
        nfcAdapter.setNdefPushMessageCallback(this, this);
        //This will be called if the message is sent successfully
        nfcAdapter.setOnNdefPushCompleteCallback(this, this);
    } else {
    	//tell user to enable NFC and return to previous activity
		...
        }
}
```

Zur Vereinfachung des hier dargestellten Codes wurde rausgelassen, was geschieht, falls das Gerät kein NFC besitzt oder es nicht aktiviert ist. 

Die Klasse implementiert zudem die Interfaces *NfcAdapter.CreateNdefMessageCallback* und *NfcAdapter.OnNdefPushCompleteCallback*. Deshalb müssen die Funktionen *createNdefMessage* und *onNdefPushComplete* enthalten sein. In ersterer Methode wird die Nachricht, die geschickt werden soll, zusammengestellt. Hierfür wird die Id des Users aus den *SharedPreferences* ausgelesen und der Nachricht hinzugefügt. 

```java
@Override
public NdefMessage createNdefMessage(NfcEvent nfcEvent) {
	//create ndef message that contains the user id, which we want to send to the other device
    String userId = Preferences.getUserId(this);
    return new NdefMessage(new NdefRecord[]{
    	createMime("text/plain", userId.getBytes(Charset.forName("UTF-8"))),
        NdefRecord.createApplicationRecord(getPackageName())
	});
}
```

NDEF ist ein standardisiertes Datenformat, das dazu verwendet wird, Informationen zwischen einem kompatiblen NFC Gerät und einem anderen NFC Gerät oder Tag auszutauschen.

Der Nutzer muss, sobald er seine Orte mit einem anderen Nutzer austauschen will, lediglich die Activity öffnen (dorthin wird er über einen Button im *FriendsFragment* geleitet) und sein Gerät and das des anderen Nutzers halten. Der zweite Nutzer muss weder in der gleichen Activity sein, noch überhaupt die App geöffnet haben. Sobald das Gerät erkannt wird, wird nämlich direkt die *AddFriendNfcActivity* geöffnet (sie befasst sich nämlich nicht zu mit dem Senden der Daten, sondern auch mit dem Empfangen). Dass dies geschehen soll, wird über einen *Intent Filter* im Android Manifest festgelegt.

```xml
<activity
	android:name=".activities.AddFriendNfcActivity"
	android:label="@string/label_AddFriendNfcActivity">
	<intent-filter>
		<action android:name="android.nfc.action.NDEF_DISCOVERED" />
    	<category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
	</intent-filter>
</activity>
```

In der *onResume* Methode wird nun der Intent empfangen und eine Funktion aufgerufen, die diesen weiterverarbeiten soll.

```java
@Override
public void onResume() {
	super.onResume();
    handleNfcIntent(getIntent());
}
```

Die Methode *handleNfcIntent* liest daraufhin die Daten (die Id des anderen Nutzers) und sendet einen Request an den Server, um die "Freundesbeziehung" zu begründen.

## Push-Benachrichtigungen

Wie bereits angedeutet, wird für die Funktionalität die Cloud Messaging Lösung von Firebase verwendet. Um die Umsetzung der Funktionalität kümmern sich zwei Klassen im Package *services*. 

Die Klasse *MyFirebaseInstanceIDService* erweitert die Firebase Klasse *FirebaseInstanceIdService*. In dem von uns angelegten Service wird die Funktion *onTokenRefresh* implementiert, die bei jeder Erst- und Neugenerierung eines FCM Tokens aufgerufen wird. Dieser Token wird dafür verwendet, um zwischen verschiedenen Geräten zu unterscheiden. Das bedeutet das ein Token für ein Endgerät steht, unabhängig vom Account des Nutzers. Da wir im serverseitigen Code wissen müssen, an welches Gerät eine Push Benachrichtigung geschickt werden soll, wird der Token bei jeder Generierung (z.B. wird ein Token neu kreiert, wenn der Nutzer die App auf seinem neuen Handy installiert) an das Backend zur Speicherung im Eintrag des Nutzers geschickt. 

Die Klasse *MyFirebaseMessagingService* erweitert die Firebase Klasse *FirebaseMessagingService* und definiert in der Methode *onMessageReceived* das Verhalten bei Empfang einer Push Benachrichtigung vom Server. Da wir ausschließlich mit *Data Messages* (nicht mit *Notification Messages*) arbeiten, wird diese Funktion bei jedem Empfang aufgerufen, unabhängig davon, ob die App im Vordergrund oder Hintergrund ist. Für eine genauere Unterscheidung der Nachrichtentypen verweisen wir an diesem Punkt auf den [Leitfaden](https://firebase.google.com/docs/cloud-messaging/android/receive) von Firebase. 

Im Manifest wird angegeben, dass diese Services bei den entsprechenden Ereignissen aufgerufen werden sollen.

```xml
<service
	android:name=".services.MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
    	<action android:name="com.google.firebase.MESSAGING_EVENT" />
	</intent-filter>
</service>
<service
	android:name=".services.MyFirebaseInstanceIdService"
    android:exported="false">
    <intent-filter>
    	<action android:name="com.google.firebase.INSTANCE_ID_EVENT" />
	</intent-filter>
</service>
```

Da wir dem Nutzer eine visuelle Benachrichtigung anzeigen wollen, wenn die App im Hintergrund ist, wird bei Empfang einer Push Notification mittels der Klasse *NotificationCombat.Builder* eine Benachrichtigung kreiert. 

Im aktuellen Stand des Produktes verwenden wir Push Benachrichtigungen an zwei Stellen, jeweils das Hinzufügen eines neuen Freundes betreffen. 

Wenn der Nutzer seine Orte in der *AddFriendEmailActivity* mit einem zweiten Nutzer teilt, so soll dieser benachrichtigt werden. Hier kommt auch die visuelle Benachrichtigung, die im letzten Absatz beschrieben wurde, ins Spiel. Diese Benachrichtigung enthält einen Button "Share your's, too", der dem zweiten Nutzer die Möglichkeit gibt direkt ebenfalls seine Orte zu teilen. Dies läuft über den dritten Service im services Package, den *NotificationActionService*, der einen *IntentService* erweitert. Hier wird ein Request an den Server gesendet durch den der Nutzer seine Orte nun auch teilt. 

Die zweite Anwendung der Push Notification Funktion von Firebase ist das Hinzufügen eines Freundes via NFC. Da uns bei der Übertragung durch NFC, ohne zu große negative Beeinflussung der User Experience, nur eine unidirektionale Übertragung möglich ist, dient uns die Möglichkeit der Push Notifications als Lösung dieses Problems. Das Initiator-Gerät der NFC Verbindung schickt die Id seines Users and das zweite Gerät. Dieses sendet nun einen Request an den Server, um die Orte seines Nutzers zu teilen. Das Backend verarbeitet diese Anfrage und schickt daraufhin eine Push Benachrichtigung mit der Id des Nutzers des zweiten Gerätes an das Initiator-Gerät, das nun wiederum einen Request zum Teilen der Orte absendet. Dies läuft jedoch alles im Hintergrund ab (hier wird im Vergleich zum vorherigen Anwendungsfall keine visuelle Benachrichtigung angezeigt), sodass die Nutzer davon nichts mitbekommen, um so eine möglichst reibungslose Nutzererfahrung zu gewährleisten. Dieses etwas kompliziert erscheinende Verfahren ist nötig, um die Sicherheit der Rest API mit einer definierten Authentifizierungsstrategie zu kompromittieren. Jeder User darf nämlich nur seine Orte mit einem anderen teilen, jedoch nicht umgekehrt. Für die Details siehe hierzu das Kapitel, das sich mit der Implementierung des Backends befasst.

## Offline-Nutzung

Die Applikation soll, zwar vorerst mit Einschränkungen, auch offline eine zufriedenstellende User Experience bieten. So wird zum Beispiel in den Fragments der *HomeActivity* abgefragt, ob der Nutzer eine Internet Verbindung besitzt. Die hierfür benötigte Methode wurde in der *Request* Klasse implementiert. Ist der Nutzer zum Beispiel offline, wird im *MyLocationsFragment* und *MyFriendsFragment* anstatt, dass dort die jeweils relevanten Daten geladen werden, eine abgeänderte Anzeige angezeigt. Diese soll den Nutzer darüber informieren, dass er gerade keinen Zugriff zum Internet besitzt. Siehe hierzu das Kapitel Design.

Des Weiteren besteht weiterhin die Möglichkeit Freunde via NFC hinzuzufügen, auch wenn einer der beiden oder beide Nutzer über keine Netzwerkverbindung verfügt. Hierzu wird die während der NFC Verbindung übertragene Id in den *SharedPreferences* zwischengespeichert. Dies funktioniert auch für mehrere neue Freunde. Beim erneuten Start der Applikation (bzw. beim Aufrufen der *HomeActivity*) wird überprüft, ob es Freunde gibt, zu denen noch eine Beziehung aufgebaut aufgebaut werden muss. Daraufhin wird ein Request an den Server geschickt, um dies zu erledigen. 

In einer nächsten Version der App wäre es vorstellbar, eine ausgeweiterte Umsetzung der Offline-Nutzung in die App zu integrieren. Eine Implementation einer vollständigen Synchronisation der Daten mit einer lokalen Datenbank (z.B. SQLite) wäre denkbar. Dadurch wäre eine gänzlich uneingeschränkten Nutzung der App auch ohne Internetverbindung gewährleistet. 


## Design

### Slogan

Die Grundidee, Backpacker zusammenzubringen und ihnen den Austausch von Empfehlungen zu erleichtern, wird im Slogan "The personal way to backpack" ausgedrückt. Dadurch, dass der Nutzer nicht mehr auf anonyme Hinweise aus Foren, Reisebüros oder Touristeninformationen angewiesen ist, sondern "aus erster Hand" erfährt, welche Orte sehenswert sind, fördert den Austausch der Nutzer untereinander, was insgesamt zu einer positiven Nutzererfahrung führen soll. 

Durch das Wort "personal" soll ausgedrückt werden, dass die App hilft, die Kommunikation über interessante Orte zu fördern, den Austausch der zugehörigen Informationen zu vereinfachen und das menschliche Bedürfnis zu befriedigen, Teil einer Gruppe zu sein. Ziel der Anwendung ist es, aus einer "anonymen" Backpacker-Reise eine persönliche zu machen, in dem man sich auf persönliche Informationen und Empfehlungen von bekannten Menschen vertraut. 

### Logo und Icons 

Schon beim ersten Hinblicken scheint der Rucksack ein Gesicht zu sein. Dieser Eindruck entsteht, weil der Reisverschluss mittig im Rucksack sitzt und die zwei Schnallen in gleichem Abstand im oberen Drittel platziert sind. Das Logo wurde einer öffentlich zugänglichen Vekorgrafik von Freepik nachempfunden (siehe <https://www.freepik.com/free-vector/flat-backpack-collection_968686.htm>). Durch das Logo soll der eindruck einer gelösten Stimmung vermittelt werden. Eine Variante des Logos mit einem "traurigen" Rucksack wird in den Fragmenten "My Locations" und "My Friends" angezeigt, falls der Nutzer keine Verbindung mit dem Internet hat und somit keine Locations/Freunde geladen werden können. 

Die Hauptfarben des Rucksacks und auch der App entsprechen einem kräftigen, satten Grün. Die Verbundenheit zur Natur wird durch die Farbwahl dargestellt. Die hohe Sättigung hat einen fröhlichen und unbekümmerten Charakter, was ganz gut zum Thema Reise und "Auszeit" passt. 

### Umsetzung in Android

Das Logo wird auch als Start-Icon für die Anwendung herangezogen. Alle anderen Icons, die in der Anwendung verwendet wurden, stammen aus dem Material Design von Android, da der Nutzer diese aus anderen Apps gewohnt ist und durch diesen Gewohnheitseffekt wiederum die Usability erhöht werden kann. Die Icons des Material Designs haben semantische Namen und werden gemäß ihrer Funktion ausgewählt, d. h. ist das Material Icon "Favorite" wird auch dafür verwendet, einen Ort zum Favoriten zu setzen. Außerdem wird die Integration dieser Icons von Android Studio sehr gut unterstützt. 

Für die Anwendung wurde in Android Studio ein neues Theme angeleget, welches die meisten Eigenschaften vom Default-Theme "App Compat Light" erbt.  Der größte Unterschied besteht in den Farben. 

Bis auf die "Activity Location Details" und das "My Map Fragment" wurden alle Layouts mit dem Constraint Layout konstruiert, da es flexible Aufteilungen der einzelnen Views ermöglicht. Im My-Map-Fragment war eine Umstellung darauf wegen der Einbindung der Karte nicht möglich. Die Activity Location Details, welche wiederum ein Fragment einbinden, wird mit einem Coordinator Layout gefüllt und enthält nur eine Appbar und einer Toolbar. Entsprechend der geladenen Daten aus dem Backend wird das Layout der Activity mit dem Fragment gefüllt. 

Beim Gestalten des ConstraintLayouts wurde versucht, weitestgehend auf direkte Formatierung zu verzichtet und Variablen für beispielsweise Abstandsregelungen oder Schriftgrößen eingeführt. So gibt es beispielsweise eine Variable "margin-normal" für den einfachen Abstand zwischen zwei Views oder Viewgroups oder ein Font-Größenangabe für Fließtext ("FloatingText"). Definiert werden die Variablenwerte im XML "Dimens.xml" im Ordner Values. Dies müsste im weiteren Verlauf der Entwicklung noch an einigen Stellen angepasst werden.

Auch Labels der Activities wurden in einer XML-Datei "strings.xml" hinterlegt, um so wenig direkten Text wie möglich zu haben. Dies würde z. B. eine spätere Übersetzung einfacher machen, welche ja für eine interantionale Zielgruppe sinnvoll wäre.  

Auch die Kategorien für die Location-Definition sind in einer XML-Datei ("") gespeichert. Wir haben uns dazu entschlossen, nicht die Kategorie-Informationen der Google-Api zu nutzen. Dies hat zum einen den Vorteil, dass  wir unsere Orte explizit auf die eingeschränkte Nutzergruppe anpassen können. Zum anderen haben wir durch die XML-Datei die Möglichkeit, die Liste schnell zu erweitern. 

Alle Layouteinstellungen wurden nur für die vertikale Ausrichtung des Smartphones festgelegt, genauer genommen wurde "sensorPortrait" eingestellt, um auch eine Ansicht zu ermöglichen, wenn der Nutzer das Handy um 180° dreht. Eine Layoutanpassung an die horizontale Ausrichtung müsste in einer zweiten Entwicklungsphase ergänzt werden. 

# Anforderungen an die Anwendung
Die Anwendung setzt die Funktionalitäten von mindestens Android 5 (API-Level 21) voraus. <!--Dies hängt vor allem auch mit der Nutzung von NFC zusammen.--> 

Laut einer Studie vom Januar 2018 liegt der Marktanteil der Android-Smartphones mit Versionen höher als 5.0 weltweit bei 80,7 %. ![Marktanteil der Android-Versionen an allen Geräten mit Android OS weltweit im Zeitraum 02. bis 08. Januar 2018 (Quelle: https://de.statista.com/statistik/daten/studie/180113/umfrage/anteil-der-verschiedenen-android-versionen-auf-geraeten-mit-android-os/, Zugriff: 24.01.2018)](C:\Users\Rebecca Durm\AndroidStudioProjects\wanderlust\res\statistik-android-apps.jpg)

### Navigation

Bei der Navigation zwischen den Listen, der Karte und den Settings in der Home-Activity haben wir uns für eine Bottom-Navigation entschieden. Dies hat folgende Gründe: Zum einen dienen die einzelnen Views vorwiegend zur Darstellung der Informationen: einer Liste über die eingene Orte, eine Liste über die Freunde und eine Karte, in der die beiden Informationen dargestellt werden. Die Menge der  Top-Level-Views ist also deshalb geringer. Dies ist vor allem im Smartphone-Bereich eine geläufige Navigationsstruktur und somit nahe am Nutzungsverhalten, welches der Nutzer auch in anderen Apps anwendet, beispielsweise bei der Smartphone-App von Youtube (Version 13) 

[^1]: Quelle:  https://material.io/guidelines/patterns/navigation.html#navigation-patterns

  Die alternative Navigation mit Tabs wurde ausgeschlossen, weil die Swipe-Gesten, die für das Navigieren zwischen den Tabs genutzt wird, auch für das View der Map genutzt wird. Es kann zu Verwirrung kommen und wird auch vom Material Design nicht empfohlen

[^2]: Quelle: https://material.io/guidelines/components/tabs.html


# Backend

Das Wanderlust-Backend ist ein Web-Service zur Verwaltung von Usern, Locations und Administratoren, sowie entsprechender Beziehungen. Es wird sowohl von der Android-App als auch von der Administations-Oberfläche verwendet. Ansprechbar ist es über eine REST-konforme Schnittstelle.

## Anleitung

Um das Backend lokal auszuführen sind folgende Schritte notwendig:

Zuerst müssen folgende Abhängigkeiten installiert werden:

* Node.js
* NPM
* MongoDB

Daraufhin kann das Backend installiert werden:

1. Das Repository klonen.
2. `npm install` ausführen.

Schließlich kann das Backend gestartet werden:

1. Stelle sicher, dass die MongoDB-Instanz läuft. 
2. Führe `npm start` aus.

Dabei wird, falls noch nicht vorhanden, ein Administrator mit dem Benutzernamen _admin_ und dem Passwort _1234_ angelegt. Die API-Dokumentation befindet sich unter  [http://localhost:3000/docs](http://localhost:3000/docs).

## Architektur

Das Design des Backends orientiert sich an folgenden Eigenschaften, die erreicht werden sollen:

- REST-Konformität
- Sicherheit
- Skalierbarkeit

Bei dem Backend handelt es sich um eine Node.js-Anwendung. Weitere wichtige Technologien werden im Laufe der Dokumentation erwähnt.

### Projektstruktur

```
├───config
├───node_modules
├───src
│   ├───middleware
│   │   └───auth
│   ├───models
│   ├───routes
│   │   ├───locations
│   │   └───users
│   └───schemas
└───uploads
    └───imgs
        ├───location
        └───user
```

Im Root-Verzeichnis befinden sich Konfigurationsdateien für Editor Config und ESLint, sowie die Swagger-Spezifikation, eine .gitignore-Datei, die package.json und package-lock.json.

Unter */config* werden alle Konfigurationsdateien abgelegt. Diese werden mit Ausnahme der *default.json* nach der Umgebung benannt, für die sie gelten sollen.

Unter */node_modules* installiert npm die in der package.json definierten Dependencies.

Im */src*-Verzeichnis befindet sich die tatsächliche Logik des Backends.

Unter */src/middleware* befindet sich die projektspezifische Middleware, also solche, die nicht über npm-Module installiert werden.

Unter */src/models* befinden sich die Mongoose-Models mit ihren Schemata. Unter */src/routes* werden die Enpunkte definiert. Im Verzeichnis */src/schemas* befinden sich Joi-Schemata für die Request-Validierung.

Eine wichtige Datei ist außerdem */src/index.js*. Sie stellt den Eintrittspunkt in die Anwendung dar und initialisiert die Express.js-Anwendung.

Im Repository befindet sich zwar kein Verzeichnis */uploads*, dieses wird aber angelegt, sobald Bilder hochgeladen werden. Hier werden alle Medien, die von Nutzern hochgeladen werden, gesichert.

### Schnittstellen

Die Schnittstellen der API werden als [OpenAPI-Spezifikation](https://github.com/OAI/OpenAPI-Specification) dokumentiert, die sich in der Datei *swagger.yml* im Root-Verzeichnis des Projekts befindet. Ein UI zu dieser Spezifikation ist unter dem Endpunkt */docs* zu erreichen, lokal also unter  [http://localhost:3000/docs](http://localhost:3000/docs). Hier können alle Informationen über die Endpoints eingesehen werden, sowie Requests ausprobiert werden. Um dieses UI bereitzustellen wird die Middleware [swagger-ui-express](https://github.com/scottie1984/swagger-ui-express) eingesetzt. Im Gegensatz zur offiziellen UI-Middleware kommt diese ohne manuell aufzulösende Abhängigkeiten aus. Da die sich die Spezifikation im Repository befindet enthält jede Version des Backends immer eine passende Dokumentation ihrer API. Eine Offline-Version dieses UI liegt der Dokumentation bei. 

| Endpoint                                 | Method | Description                              |
| ---------------------------------------- | ------ | ---------------------------------------- |
| `/users`                                 | GET    | Find users by criteria specified as query parameters. Admins can omit those criteria, users have to specify them. |
| `/users`                                 | POST   | Create user. Users can only create profiles linked to their own google account. |
| `/users/{userId}`                        | GET    | Get user. Users can only access their own profile. |
| `/users/{userId}`                        | PATCH  | Update user. Users can only update their own profile. |
| `/users/{userId}`                        | DELETE | Delete user. Users can only delete their own profile. |
| `/users/{userId}/friends`                | GET    | Get user's friends. Users can only access their own friends. |
| `/users/{userId}/friends/{friendId}`     | PUT    | Add friend. This will share the user's locations with the added friend. The friend does not automatically share their locations with the user. Users can only share their own locations. |
| `/users/{userId}/friends/{friendId}`     | DELETE | Remove friend.                           |
| `/users/{userId}/avatar`                 | PUT    | Upload avatar                            |
| `/users/{userId}/avatar`                 | DELETE | Remove Avatar                            |
| `/locations`                             | GET    | Get the locations of the users specified as query parameters |
| `/locations`                             | POST   | Add new location                         |
| `/locations/{id}`                        | PATCH  | Update location                          |
| `/locations/{id}`                        | DELETE | Delete Location                          |
| `/locations/{id}/images`                 | POST   | Upload location images                   |
| `/location/{locationId}/images/{imageId}` | DELETE | Delete Image                             |

Definiert werden die Schnittstellen mithilfe von [Express.js](https://expressjs.com/). Die entsprechenden Routen befinden sich im Verzeichnis */src/routes*. Bei der Definition der Endpunkte wird großer Wert auf REST-Konformität gelegt. Die HTTP-Methoden werden dabei so genutzt wie es ihre Spezifikationen vorsehen:

**GET** wird verwendet um Ressourcen vom Backend anzufordern. Es werden dabei keine Ressourcen geändert.

**POST** wird verwendet um neue Ressourcen anzulegen. Dabei legt der Server fest, unter welcher URL die Ressource erreichbar sein wird. Die Daten werden über den Request-Body übermittelt. Wiederholtes Senden desselben Requests führen dabei jedes Mal zu einem anderen Ergebnis, da dieselbe Ressource unter verschiedenen IDs abgespeichert wird.

**PUT** hingegegen wird verwendet, wenn eine Ressource an der durch die URL definierte Stelle erzeugt werden soll. Damit können neue Ressourcen angelegt werden, aber auch komplette Ressourcen ausgetauscht werden.

**PATCH** kann verwendet werden, wenn Änderungen an einer Ressource vorgenommen werden sollen, ohne die gesamte Ressource auszutauschen. Dabei werden die Daten im Format [JSON Merge Patch](https://tools.ietf.org/html/rfc7386) übergeben. Dieses Format hat einige Nachteile gegenüber dem [JSON Patch](https://tools.ietf.org/html/rfc6902)-Format. So müssen beispielsweise um Daten zu Arrays hinzuzufügen extra Subroutes angelegt werden. Dafür ist der Request intuitiver und einfacher zu verarbeiten. Der Content-Type-Header sollte auf *application/merge-patch+json* gesetzt werden um explizit auszudrücken, dass dieses Format verwendet wird. Dadurch wird auch die Möglichkeit offen gehalten in Zukunft andere Formate zu unterstützen, ohne die Backwards-Compability der API zu brechen.

**DELETE** wird verwendet um Ressourcen zu löschen.

### Modelle

Für die Datenhaltung kommt [MongoDB](https://www.mongodb.com/) zum Einsatz. Diese wird durch [Mongoose](http://mongoosejs.com/) in das Backend eingebunden, wodurch erweiterte Funktionalitäten zur Verfügung stehen und - besonders wichtig - Schemas definiert werden können. Im Backend gib es drei Datenmodelle: Admin, User und Location. Im Verzeichnis */src/models* werden diese Modelle definiert.

#### Admin

| field    | type   | required | default | ref  |
| -------- | ------ | -------- | ------- | ---- |
| username | String | true     | -       | -    |
| password | String | true     | -       | -    |

Eine Besonderheit der Admin-Collection ist, dass dort auch Passwörter gehalten werden. Dies kann nicht in Form von Klartext geschehen. Vor dem Speichern der Paswörter muss also ein Einweg-Hash erzeugt werden, gegen das dann andere Passwörter verifiziert werden können. Dies geschieht im _pre save_ Hook des Schemas. Zur Generierung des Salt-Wertes und des Hashes wir die Bibliothek [bcrypt](https://github.com/kelektiv/node.bcrypt.js) verwendet. Außerdem wird das Schema um eine Methode erweitert, mit welcher Passwörter validiert werden können. Auch hierfür kommt *bcrypt* zum Einsatz.

#### User

| field       | type       | required | default | ref      |
| ----------- | ---------- | -------- | ------- | -------- |
| googleId    | String     | true     | -       | -        |
| deviceToken | String     | false    | -       | -        |
| firstName   | String     | true     | -       | -        |
| lastName    | String     | true     | -       | -        |
| email       | String     | true     | -       | -        |
| locations   | [ObjectId] | false    | -       | Location |
| friends     | [ObjectId] | false    | -       | User     |
| avatar      | String     | false    | -       | -        |

Da jede Location einem User zugeordnet ist und ein User mehrere Locations besitzen kann, muss darauf geachtet werden, dass beim Erzeugen der Location, die entsprechende ID im User-Dokument eingetragen wird und dass diese beim Löschen der Location wieder entfernt wird. Beim Hinzufügen eines Freundes muss darauf geachtet werden, dass die Freunde eines Users ausdrücken, welche anderen User ihre Orte für ihn freigegeben haben. Wenn man eine gegenseitige Freundschaft anlegen will, müssen beide Freundeslisten erweitert werden. Um Freundschaften zu entfernen müssen beide Nutzer aus der jeweilig anderen Liste entfernt werden.

#### Location

| field       | type       | required | default | ref  |
| ----------- | ---------- | -------- | ------- | ---- |
| name        | String     | true     | -       | -    |
| user        | ObjectId   | true     | -       | User |
| description | String     | false    | ''      | -    |
| favorite    | Boolean    | false    | false   | -    |
| googleId    | String     | true     | -       | -    |
| coordinates | [Number]   | true     | -       | -    |
| city        | String     | true     | -       | -    |
| country     | String     | true     | -       | -    |
| images      | [ObjectId] | false    | -       | -    |
| categories  | [String]   | false    | -       | -    |

## Authentifizierung und Autorisierung

Die Endpunkte werden so gesichert, dass sie nur von registrierten Nutzern und Administratoren angesprochen werden können (Authentifizierung). Aber auch angemeldete Nutzer dürfen nicht jeden der Endpunkte mit beliebigen Parametern ansprechen (Autorisierung). Die gesamte Authentifzierung und Autorisierung geschieht in den Middleware-Funktionen, die sich unter */src/middleware/auth* befinden. Die Authentifizierung geschieht dabei mithilfe der Middleware Passport.js. Darauf baut dann die Autorisierung auf.

Eine wichtige Middleware-Funktion ist dabei die Methode `init`. In dieser werden Passport.js und verschiedene Strategien initialisiert. Es wird außerdem ein Default-Admin angelegt, falls dieser noch nicht existiert. Dabei wird eine Strategie verwendet um Nutzer anhand ihres Google Tokens zu identifizieren. Dafür wird die Strategie [GoogleIdToken](https://github.com/jmreyes/passport-google-id-token) verwendet. Um Administratoren anhand von Nutzernamen und Passwort zu authentifizieren wird die Strategie [BasicAuthentication](https://github.com/jaredhanson/passport-http) verwendet. Für beide Strategien müssen `verify`-Funktionen implementiert werden. In diesen Funktionen wird überprüft ob ein solcher Nutzer existiert und gegebenenfalls ob das Passwort korrekt ist. Anschließend wird dann dieser Nutzer auf ein User-Objekt gemapt, das dann an das Request-Objekt angehängt wird, sodass in folgender Middleware darauf zugegriffen werden kann.

Strategien, die dafür genutzt werden Endanwender zu authentifizieren können diese Nutzer auch auf Objekte mit denselben Eigenschaften mappen. Das user-Objekt muss aber auch Administratoren repräsentieren können, sowie Nutzer, die zwar einen gültigen Google-Account haben, sich aber noch nicht für Wanderlust registriert haben. Deshalb werden die User-/Admin-Dokumente aus der Datenbank um ein `type`-Attribut erweitert. Für User, die noch nicht in der Datenbank zu finden sind wird ein Objekt verwendet, das nur dieses `type`-Attribut sowie die googleId besitzt. Die drei verschiedenen User-Types ermöglichen in folgender Middleware die Autorisierung basierend auf den Type sowie der Eigenschaften des Users.

Folgende Typen werden dabei verwendet:

- user_candidate: Es handelt sich um einen validen Google User, allerdings wurde er noch nicht für Wanderlust registriert. Dieser type spielt nur für die Registrierung neuer Nutzer eine Rolle.
- user: Es handelt sich um einen validen und für Wanderlust registrierten Google User. Für alle Interaktionen der App mit der API ist dies der Fall, mit Ausnahme der Nutzer-Registrierung.
- admin: Es handelt sich um einen Administrator, der sich erfolgreich mit Nutzernamen und Passwort authentifiziert hat. Ein Administrator kann alle Endpunkte ansprechen und hat meehr Freiheiten, was die Wahl der einzelnen Parameter betrifft. API-Anfragen mit Administratoren-Rechten kommen normalerweise aus dem Frontend, können aber auch über die Swagger UI durchgeführt werden.

Um eine Route mithilfe der Authentifizierung zu schüzten, wird die Middleware `authenticate` verwendet. Diese stellt einen einfachen Wrapper um die `authenticate`-Middleware von Passport.js dar. Sie deaktiviert dabei für alle Aufrufe Sessions, da diese die Zustandslosigkeit der API beeinträchtigen.

Die anderen Middleware-Funktionen im Verzeichnis */src/middleware/auth* können dann zur Autorisierung verwendet werden. Mit diesen kann festgelegt werden, welche User-Types Zugriff auf die Route bekommen sollen, sowie Anforderungen an die Request-Parameter definiert werden. Dabei wird immer ein Pfad angegeben unter dem der Parameter im Request-Objekt zu finden ist, sodass dieselbe Middleware verwendet werden kann um den Request-Body sowie Query-Parameter und URL-Parameter zu überprüfen.

Middleware, die verwendet wird um Request-Parameter zu überprüfen, sollte immer nach der Celebrate-Middleware verwendet werden, da diese zum einen die Request-Parameter noch verändern kann, zum anderen aber auch alle Parameter vorhanden sind, die von der Middleware erwartet werden.

## Datei-Uploads

Bilder werden als [multipart/form-data](multipart/form-data) übertragen. Um diese serverseitig verarbeiten zu können wird das Modul [formidable](https://github.com/felixge/node-formidable) verwendet. Da jeder Nutzer nur ein Profilbild besitzt können als Dateinamen die IDs der Nutzer verwendet werden. Für Location-Bilder werden neue IDs generiert, welche dann als Dateiname verwendet werden und in der Location gesichert werden.

Profilbilder werden unter */uploads/imgs/user* abgelegt, Bilder zu Locations werden unter */uploads/omgs/location* gespeichert. Um die Anwendung skalierbar zu machen, müsste der interne Speicher durch einen File Store Server ersetzt werden.

## Notifications

Um Notifications an Geräte zu versenden wird das offizielle [Firebase Admin](https://firebase.google.com/docs/reference/admin/node/)-Modul verwendet. Unter Verwendung des in der User-Collection gespeicherten Device-Tokens, können damit Benachrichtigungen an individuelle Geräte gesendet werden.

## Request-Validierung

Um fehlerhafte oder boshafte Requests an die API zu verhindern müssen alle Requests validiert werden. Fehlerhafte Query-Parameter, URL-Parameter oder Body-Parameter müssen zu einem `400` Status Code führen, oder falls möglich die Daten auf die richtige Form gebracht werden. Solch ein Whitelisting sorgt dafür, dass Request-Daten direkt weiterverarbeitet werden können, ohne die Integrität der Daten zu gefährden.

Für diese Validierung bedienen wir uns bei [Hapi](https://github.com/hapijs/hapi), einer von Walmart entwickelten Alternative zu Express. Dieses beinhaltet die Bibliothek [Joi](https://github.com/hapijs/joi), mit welcher Validierungsschemas beschrieben werden können. Um Joi in Express nutzen zu können, kommt die Middleware [celebrate](https://github.com/continuationlabs/celebrate) zum Einsatz.

## Konfiguration

Die Konfiguration der Anwendung geschieht über die Bibliothek [config](https://github.com/lorenwest/node-config). 

Das Backend lässt sich über Konfigurationsdateien im Verzeichnis `/config` konfigurieren. Dabei können für jede Umgebung eigene Konfigurationsdateien erstellt werden, die dann entsprechende Werte aus der Datei `default.json` überschreiben. Dabei muss nur darauf geachtet werden, den Namen der Konfiguration  entsprechend der gewünschten Umgebung zu benennen. Sie wird dann verwendet, wenn dieser Umgebungs-Name als `NODE_ENV`-Umgebungsvariable gesetzt wird. Da wir das Backend bisher noch nicht deployed haben, ist bisher nur die `default.json` vorhanden.

## Logging

Alle Requests, die vom Backend verarbeitet werden, werden geloggt. Dafür kommt die Middleware [morgan](https://github.com/expressjs/morgan) zum Einsatz. Das Format kann dabei über `logging.format` in der Konfigurationsdatei des Backends angepasst werden, sodass für verschiedene Stages verschiedene Formate definiert werden können. Morgan unterstützt dabei sowohl vordefinierte auch benutzerdefinierte Formate.

Zusätzlich werden auch bestimmte Informationen, wie der verwendete Port und die verwendete Datenbank geloggt, sowie verschiedene Server-Fehler. Dabei wird [chalk](https://github.com/chalk/chalk) verwendet, um die Logs farblich passend hervorzuheben.

## JavaScript Style Guide

Um konsistenten und hochwertigen Code zu schreiben verwendet das Backend (wie auch das Frontend) den [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript). Dieser beschreibt verschiedene Regeln, wie Code geschrieben wird. Um die Einhaltung dieser Regeln zu erzwingen, verwenden wir [ESLint](https://github.com/eslint/eslint). Dieses kann mithilfe von `npm run lint` ausgeführt werden. Um zu verhindern, dass vor einem Commit vergessen wird, dieses Skript auszuführen (wodurch inkonsistenter Code im Repository landen könnte) ist dieses Skript am Pre-Commit-Hook registriert. Das bedeutet, dass vor jedem commit die Einhaltung des Style Guides überprüft wird und nur im Erfolgsfall der tatsächliche Commit ausgeführt wird.

## Ausblick

Während das Backend in seiner derzeitigen Form funktionstüchtig ist und auch Sicherheit und REST-Konformität gewährleistet, gibt es noch Ansatzpunkte, welche das Backend verbessern könnten.

### Authentifizierung über Authorization-Header

[RFC 7235](https://tools.ietf.org/html/rfc7235) definiert den Authorization-Header als den passenden Header um Credentials mit einem Request zu verschicken. Für Basic Authentication wird dieser Header in unserem Projekt auch verwendet, die Passport.js-Strategie, mit welcher wir die Google-Authentifizierung umsetzen, berücksichtigt allerdings den Authorization-Header nicht. Stattdessen muss ein seperater Custom Header verwendet werden. Wir haben deshalb die Bibliothek um die gewünschte Funktionalität erweitert, sodass sie auch Bearer Token, die sich im Authorization Header befinden, berücksichtigt. Der entsprechende [Pull Request](https://github.com/jmreyes/passport-google-id-token/pull/26) muss allerdings noch bestätigt werden. Sobald dies der Fall ist kann das Projekt entsprechend angepasst werden, sodass jede Form von Authentifizierung über den Authorization Header abläuft.

### Response-Validierung

Bisher wird Joi verwendet um Requests zu validieren und damit unerwünschte Requests zu unterbinden. Auf dieselbe Weise könnten auch Responses validiert werden, um beispielsweise zu verhindern, dass sensible Daten aufgrund eines Fehlers in einem Select-Statement oder durch eine ausgegebene Fehlermeldung preisgegeben werden. Inwiefern dies durch `celebrate`, der Middleware, die Joi in unser Projekt einbindet, unterstützt wird, müsste noch untersucht werden.

### File Store

Bisher speichern wir Bilder von Orten und Endanwendern auf dem Dateisystem des Servers ab, auf dem die Anwendung ausgeführt wird. Da angedacht ist mehrere Instanzen des Backends zu deployen, müssten die Bilder tatsächlich auf einem File Store Server abgespeichert werden.

### Deployment

Um das Backend in Produktion einzusetzen, muss es deployed werden. Um mit wechselnder Last zurecht zu kommen, müssten dynamisch mehrere Instanzen der Anwendung erzeugt werden. Da unsere Anwendung bis auf den oben schon genannten fehlenden File Storage zustandslos ist, lässt sie sich beliebig skalieren. Dafür müsste nur ein Load Balancer vorgeschaltet werden.

Um das Backend in Produktion einzusetzen müsste noch eine neue Konfigurationsdatei mit dem Namen production.json angelegt werden und auf dem Server die Umgebungsvariable `NODE_ENV=production` gesetzt werden.

### Google Token Validation

Jeder Request, der einen Google-Token mitschickt, wird über eine Schnittstelle von Google verifiziert. Um die Netzwerklast zu reduzieren könnten diese Anfragen gecached werden. Dafür müsste eine `getGoogleCerts`-Funktion an die Strategie übergeben werden, welche dieses Verhalten umsetzt.

# Administrations-Oberfläche

Die Administrationsoberfläche bietet Administratoren die Möglichkeit, Daten der Nutzer und Orte einzusehen und zu bearbeiten. So können beispielsweise Nutzer gelöscht werden, Freundschaften beendet werden oder Profile bearbeitet werden.

Es handelt sich dabei um eine Vue.js-Anwendung. Die Anwendung wurde mit dem [vue-cli](https://github.com/vuejs/vue-cli) erzeugt. Als Template wurde [webpack](https://github.com/vuejs-templates/webpack) verwendet.

```
├───build
├───config
├───node_modules
├───src
│   ├───assets
│   ├───components
│   └───router
└───static
```

Die Projektstruktur wird auf der Webseite des Templates [dokumentiert](https://vuejs-templates.github.io/webpack/structure.html).

Das Template befindet sich im Route-Verzeichnis und trägt den Namen *index.js*. Hier werden beispielsweise HTML-Header festgelegt. Die restlichen relevanten Dokumente befinden sich im */src*-Verzeichnis. Die zentrale Vue-Komponente heißt App.vue und befindet sich direkt im */src*-Verzeichnis. Sie verwendet den unter */src/router* definierten Router. Dieser verweist bisher nur auf eine einzige weitere Komponente, der Users-Komponente. Diese befindet sich im Verzeichnis */src/components*. Sie beschreibt eine Tabelle, mit allen Nutzern im System. Sie bietet die Möglichkeit User zu löschen, sowie Buttons, mit denen sich Modals öffnen lassen, über die dann Nutzerdaten bearbeitet werden können, sowie Freunde und Locations gelöscht werden können. Diese Modals befinden sich in eigenen Komponenten im selben Verzeichnis. Die Kommunikation zwischen diesen Komponenten und der Users-Komponente geschieht über Props und Events.

Um [Bootstrap](https://getbootstrap.com/) als Vue.js-Komponente in das Projekt einzubinden wurde [bootstrap-vue](https://github.com/bootstrap-vue/bootstrap-vue) verwendet. Um [Font Awesome](http://fontawesome.io/)-Icons zu verwenden kommt außerdem [vue-awesome](https://github.com/Justineo/vue-awesome) zum Einsatz. 

## API-Requests

Um mit dem Backend zu kommunizieren wird [axios](https://github.com/axios/axios) verwendet. Entsprechende Controller befinden sich in der Datei */src/api.js*.

## Ausblick

Im Gegensatz zu der Android-Anwendung und dem Backend ist die Administrationsoberfläche noch nicht einsatzbereit. Folgende Features sollten noch umgesetzt werden:


* Log-In
* Authentifizierung
* Locations bearbeiten
* Bilder verwalten

# Ausblick

## Offline-Verhalten

In einem so speziellen Nutzungszenario wie das beim Backpacking wäre eine umfangreiche Offlinezugänglichkeit von Nöten. Im jetzigen Stand der Entwicklung wird abgefangen, ob der Nutzer online ist. Falls er es nicht ist, werden in den Fragmenten "MyLocation" bzw. "My Friends" eine Ansicht mit "You're not connected to the internet" und dem traurigen Rucksack eingeblendet. 

Das Hinzufügen eines Freundes über NFC funktioniert schon offline, in dem die auszutauschenden User-Ids im Shared-Preferences-Speicher des Smartphones gespeichert werden. Der Request an die Api-Schnittstelle startet, wenn der Nutzer die Home-Activity aufruft und wieder eine Internetverbindung besteht. Nach erfolgreichem Senden des Requests wird die String-Variable in den Shared Preferences gelöscht. 

Wünschenswert wäre eine Offlinespeicherung aller Daten auf dem Gerät. Dies hätte den Vorteil, dass alle Locations bzw. Freunde auch offline angezeigt werden können. In diesem Zusammenhang müsste auch eine geeignete Datenbank angelegt werden können, die die Daten aus MongoDB in der entsprechenden Struktur speichert. Abhängig von der Verbindung würden entweder die Daten aus dem Backend nachgeladen und aktualisiert oder die zwischengespeicherten Daten angezeigt werden. Beispielsweise könnte man sich über die Umsetzung mit einer SQLite-Datenbank Gedanken machen. 

## Pushbenachrichtigungen beim Teilen der Location-Listen

Bisher wird eine Push-Benachrichtigung geschickt, wenn der Nutzer seine Locations via Email einem anderen Nutzer freigegeben hat. Falls der Nutzer diese Nachricht wegwischen würde, ohne auf "I want to share my locations too" zu drücken, hat er momentan keine Möglichkeit, dies nachträglich zu tun. 

In der weiteren Entwicklung würde man die Push-Benachrichtungen ausweiten und Funktionalitäten so anpassen, dass innerhalb der App eine Sektion zeigt, mit wem man seine Locations geteilt hat, wer davon sich auch in meiner Freundesliste befindet und die Anfragenverwaltung, ob die Freigabe zu einem späteren Zeitpunkt erfolgen soll. 

## Chatfunktion

Bisher ist es nicht angedacht, dass die Nutzer außerhalb des Location-Austausches miteinander kommunizieren. In Zukunft könnte geprüft werden, ob auch eine Chatfunktion eine Option für die Anwendung wäre, um mit den Leuten in Kontakt zu bleiben, die sich auf einer Reise kennengelernt haben. Die gebildete Community hätte ein gemeinsames Interessensgebiet und würde ein eigenes Netzwerk bilden. Dies könnte wiederum dazu führen, dass sich Backpacker über die Anwendung verabreden. 

## Beschränkung der Favoriten

Wanderlust hebt sich von normalen Bewertungsportalen dadurch ab, dass eine Reizüberflutung vermieden wird und nur wenige Orte pro Stadt vorgeschlagen werden. Deshalb kann es sinnvoll sein, die Anzahl der Favoriten pro Stadt zu begrenzen. Dabei könnte ein Maximum von 5 Favoriten pro Stadt sinnvoll sein. Schließlich ist die Anwendung für Backpacker gedacht und diese zeichnen sich dadurch aus, dass sie sich sehr kurz an vielen Orten aufhalten.

## Weitere mögliche Features

Es gibt einige Aktivitäten, über die man Nutzer noch mit Push-Benachrichtigungen informieren könnte. So könnte ein Nutzer darauf hingewiesen werden, dass er sich in der Nähe eines Favoriten eines Freundes aufhält. 

Ein weiteres mögliches Feature wäre das Teilen des aktuellen Standortes mit seinen Freunden.

Auch die Kartenansicht könnte erweitert werden, beispielsweise um Filter für verschiedene Kategorien.