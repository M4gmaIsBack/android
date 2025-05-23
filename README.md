ajout Meteoapp/app/build.gradle.kts

`dependencies {`
    `implementation("com.squareup.retrofit2:retrofit:2.9.0")`
    `implementation("com.squareup.retrofit2:converter-gson:2.9.0")`
    `implementation("com.squareup.okhttp3:logging-interceptor:4.10.0")`
`}`

#🔹 `implementation("com.squareup.retrofit2:retrofit:2.9.0")`

> **But :** Ajouter la librairie **Retrofit**, qui sert à faire des **requêtes HTTP** simplement (GET, POST…).


---

#🔹  `implementation("com.squareup.retrofit2:converter-gson:2.9.0")`

> **But :** dire à Retrofit comment **convertir le JSON en objets Kotlin** automatiquement.

💡 WeatherAPI te renvoie du JSON comme :

```json
{ "location": { "name": "Paris" }, "current": { "temp_c": 20.5 } }
```

Grâce à cette ligne, Retrofit convertit ça **directement** en classes Kotlin comme :

```kotlin
data class WeatherResponse(val location: ..., val current: ...)
```

---

#🔹  `implementation("com.squareup.okhttp3:logging-interceptor:4.10.0")`

> **But :** ajouter un **log des requêtes HTTP** dans la console (très utile pour débugger).

```
GET https://api.weatherapi.com/v1/current.json?q=Paris...
Response: 200 OK
```

### 🔹 `id("org.jetbrains.kotlin.kapt")`

> 📌 **KAPT = Kotlin Annotation Processing Tool**

Cela active **l’annotation processing** pour Kotlin. C’est indispensable si tu utilises des bibliothèques qui génèrent du code à la compilation à partir d’annotations, comme :

- `Room` (pour les bases de données),
    
- `Dagger/Hilt` (injection de dépendances),
    
- `Glide` (pour le cache d’images optimisé),
    
- `Moshi`, etc.
    

💡 Sans `kapt`, ces bibliothèques ne peuvent pas fonctionner en Kotlin (même si elles marcheraient en Java avec `annotationProcessor`).

---

### 🔹 `implementation("com.github.bumptech.glide:glide:4.16.0")`

> 📌 Ajoute **Glide** dans ton projet.

Glide est une **bibliothèque de chargement d’images rapide et optimisée**.

Tu peux faire :

kotlin

CopierModifier

`Glide.with(this).load(url).into(imageView)`

➡️ Elle gère automatiquement :

- le téléchargement de l’image,
    
- la mise en cache,
    
- l’affichage optimisé dans les `ImageView`.
    

---

### 🔹 `kapt("com.github.bumptech.glide:compiler:4.16.0")`

> 📌 Ajoute le **compiler Glide** pour permettre à Glide de générer du code utile à la compilation (ex: `GlideApp`).

Sans cette ligne, tu ne pourrais pas utiliser certaines fonctions avancées (ex : les extensions de Glide ou les modules personnalisés).

---

|Ligne|Sert à|
|---|---|
|`retrofit`|Faire les appels HTTP|
|`converter-gson`|Convertir JSON → Kotlin|
|`logging-interceptor`|Voir les requêtes dans la console|


---

➤ `WeatherApiService.kt`
`app/src/main/java/com/example/meteoapp/api/`

```bash
package com.example.meteoapp.api

import retrofit2.http.GET
import retrofit2.http.Query
import retrofit2.Call

// Représente la réponse principale de l'API /forecast.json
// Elle contient les infos de localisation, météo actuelle et les prévisions

// Objet racine de la réponse JSON
// "location" = où, "current" = météo actuelle, "forecast" = prévisions complètes
data class ForecastResponse(
    val location: Location,
    val current: Current,
    val forecast: Forecast
)

// Partie "forecast" contenant une liste de jours prévus
data class Forecast(
    val forecastday: List<ForecastDay>
)

// Chaque jour contient une date, les données globales et une liste d'heures
// "hour" contient 24 éléments (1 par heure)
data class ForecastDay(
    val date: String,
    val day: Day,
    val hour: List<Hour>
)

// Données moyennes/journalières (min, max, moyenne + condition)
data class Day(
    val maxtemp_c: Float,
    val mintemp_c: Float,
    val avgtemp_c: Float,
    val condition: Condition
)

// Données pour une heure précise
data class Hour(
    val time: String,
    val temp_c: Float,
    val condition: Condition
)

// Localisation (ville et pays)
data class Location(val name: String, val country: String)

// Température actuelle et condition
// → identique à WeatherResponse
// Cela permet de le mutualiser dans les 2 endpoints

// Partie "current" de l'API
// Ex : temp_c = 19.5, condition = "Sunny"
data class Current(val temp_c: Float, val condition: Condition)

// Description et icône de la météo
// Ex : text = "Partly cloudy", icon = "//cdn.weatherapi.com/..."
data class Condition(val text: String, val icon: String)

// Interface Retrofit pour interroger le endpoint /forecast.json
interface WeatherApi {

    // Appelle l'URL suivante :
    // https://api.weatherapi.com/v1/forecast.json?key=...&q=...&days=7

    @GET("forecast.json")
    fun getForecast(
        @Query("key") cle: String,              // Clé d'API personnelle
        @Query("q") requete: String,            // Ville ou coordonnées (ex: Paris ou 48.85,2.35)
        @Query("days") jours: Int = 7,          // Nombre de jours de prévision (par défaut: 7)
        @Query("aqi") aqi: String = "no",       // Désactive la qualité de l'air
        @Query("alerts") alerts: String = "no"  // Désactive les alertes météo
    ): Call<ForecastResponse>                    // Retrofit convertira ça en ForecastResponse
}

```


---
`RetrofitInstance.kt`
`app/src/main/java/com/example/meteoapp/api/`

```bash
package com.example.meteoapp.api

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

# C'est un singleton Kotlin accessible partout, on initialise Retrofit une seule fois
object RetrofitInstance {
	# lazy Retrofit sera creer que lors du premier appel 
    val api: WeatherApi by lazy {
        Retrofit.Builder()
	        # base url des requete api
            .baseUrl("https://api.weatherapi.com/v1/")
            # Ajoute la conversion automatique JSON → Kotlin
            .addConverterFactory(GsonConverterFactory.create())
            # Creer l'instance Retrofit
            .build()
            # Lie Retrofit à ton interface WeatherApi (celle avec getWeather (...)
            .create(WeatherApi::class.java)
    }
}
```

grace a ca on peut appeler directement 

`RetrofitInstance.api.getWeather("ta_cle_api", "Paris")`

---
MainActivity

```bash
package com.example.meteoapp

import android.content.Intent
import android.os.Bundle
import android.text.SpannableStringBuilder
import android.text.style.ImageSpan
import android.widget.Button
import android.widget.ImageView
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import com.bumptech.glide.Glide
import com.example.meteoapp.api.ForecastResponse
import com.example.meteoapp.api.RetrofitInstance
import retrofit2.Call
import retrofit2.Callback
import retrofit2.Response

class MainActivity : AppCompatActivity() {

    // Déclaration des vues de l'interface
    private lateinit var txtVille: TextView
    private lateinit var txtTemp: TextView
    private lateinit var txtJournee: TextView
    private lateinit var txtSemaine: TextView
    private lateinit var btnVoirListe: Button
    private lateinit var imgIconeMain: ImageView

    // Appelé au lancement de l'activité
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main) // Charge le layout XML

        // Initialisation des vues
        txtVille = findViewById(R.id.txtVille)
        txtTemp = findViewById(R.id.txtTemp)
        txtJournee = findViewById(R.id.txtJournee)
        txtSemaine = findViewById(R.id.txtSemaine)
        btnVoirListe = findViewById(R.id.btnVoirListe)
        imgIconeMain = findViewById(R.id.imgIconeMain)

        // Action lors du clic sur le bouton "Voir la liste des villes"
        btnVoirListe.setOnClickListener {
            val intent = Intent(this, ListeVillesActivity::class.java)
            startActivity(intent)
        }

        // Affiche la météo de la ville favorite au démarrage
        afficherVilleFavorite()
    }

    // Appelé quand on revient sur cette activité
    override fun onResume() {
        super.onResume()
        afficherVilleFavorite()
    }

    // Récupère la ville favorite depuis les préférences et affiche les données
    private fun afficherVilleFavorite() {
        val prefs = getSharedPreferences("meteo_prefs", MODE_PRIVATE)
        val ville = prefs.getString("ville_favorite", null)

        if (ville != null) {
            txtVille.text = ville // Affiche le nom de la ville
            appelerApiMeteo(ville) // Appelle l'API pour les infos
        } else {
            // Affiche un message par défaut
            txtVille.text = "Aucune ville favorite"
            txtTemp.text = ""
            txtJournee.text = ""
            txtSemaine.text = ""
        }
    }

    // Fait appel à l'API WeatherAPI pour récupérer les données de météo
    private fun appelerApiMeteo(ville: String) {
        val call = RetrofitInstance.api.getForecast("1f9cb2ae04d44d21a9780228252305", ville)

        call.enqueue(object : Callback<ForecastResponse> {
            override fun onResponse(call: Call<ForecastResponse>, response: Response<ForecastResponse>) {
                if (response.isSuccessful) {
                    val meteo = response.body()
                    val currentTemp = meteo?.current?.temp_c
                    val condition = meteo?.current?.condition?.text
                    val icone = meteo?.current?.condition?.icon

                    // Affiche la température actuelle + état du ciel
                    txtTemp.text = "$currentTemp°C - $condition"

                    // Affiche l'icône météo via Glide
                    Glide.with(this@MainActivity)
                        .load("https:$icone")
                        .into(imgIconeMain)

                    // Récupère les prévisions à 8h, 14h, 20h
                    val heures = meteo?.forecast?.forecastday?.get(0)?.hour
                    val matin = heures?.get(8)
                    val aprem = heures?.get(14)
                    val soir = heures?.get(20)

                    // Texte pour les prévisions de la journée avec emojis dynamiques
                    val journee = StringBuilder()
                    journee.append("Matin : ${matin?.temp_c}°C ${meteoIcone(matin?.condition?.text)}\n")
                    journee.append("Après-midi : ${aprem?.temp_c}°C ${meteoIcone(aprem?.condition?.text)}\n")
                    journee.append("Soir : ${soir?.temp_c}°C ${meteoIcone(soir?.condition?.text)}")
                    txtJournee.text = journee.toString()

                    // Prévisions pour les 7 jours suivants avec icônes
                    val jours = meteo?.forecast?.forecastday
                    val semaine = jours?.joinToString("\n") { day ->
                        val date = day.date
                        val tempMoyenne = day.day.avgtemp_c
                        "$date : $tempMoyenne°C ${meteoIcone(day.day.condition.text)}"
                    }
                    txtSemaine.text = semaine
                } else {
                    txtTemp.text = "Erreur API"
                }
            }

            override fun onFailure(call: Call<ForecastResponse>, t: Throwable) {
                txtTemp.text = "Erreur : ${t.message}"
            }
        })
    }

    // Fonction qui retourne un emoji météo en fonction de la description
    private fun meteoIcone(description: String?): String {
        return when {
            description == null -> ""
            description.contains("sun", true) || description.contains("clear", true) -> "☀️"
            description.contains("cloud", true) -> "⛅"
            description.contains("rain", true) || description.contains("drizzle", true) -> "🌧️"
            description.contains("thunder", true) -> "⚡"
            description.contains("snow", true) -> "❄️"
            description.contains("mist", true) || description.contains("fog", true) -> "🌫️"
            else -> "🌈"
        }
    }
}

```

---
MeteoActivity

```bash
package com.example.meteoapp

import android.os.Bundle
import android.widget.Button
import android.widget.ImageView
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import com.bumptech.glide.Glide
import com.example.meteoapp.api.ForecastResponse
import com.example.meteoapp.api.RetrofitInstance
import retrofit2.Call
import retrofit2.Callback
import retrofit2.Response

class MeteoActivity : AppCompatActivity() {

    // Déclaration des éléments graphiques
    private lateinit var txtVille: TextView
    private lateinit var txtTemp: TextView
    private lateinit var txtJournee: TextView
    private lateinit var txtSemaine: TextView
    private lateinit var btnSetFavorite: Button
    private lateinit var imgIcone: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_meteo) // Lien avec le fichier XML

        supportActionBar?.setDisplayHomeAsUpEnabled(true) // Active la flèche retour

        // Liaison des vues avec leur ID
        txtVille = findViewById(R.id.txtVille)
        txtTemp = findViewById(R.id.txtTemp)
        txtJournee = findViewById(R.id.txtJournee)
        txtSemaine = findViewById(R.id.txtSemaine)
        btnSetFavorite = findViewById(R.id.btnSetFavorite)
        imgIcone = findViewById(R.id.imgIcone)

        // Récupère le nom de la ville passée via l'intent ou met "Paris" par défaut
        val ville = intent.getStringExtra("VILLE") ?: "Paris"
        txtVille.text = ville // Affiche la ville

        // Bouton pour définir la ville comme favorite
        btnSetFavorite.setOnClickListener {
            val prefs = getSharedPreferences("meteo_prefs", MODE_PRIVATE)
            prefs.edit().putString("ville_favorite", ville).apply()
            Toast.makeText(this, "$ville définie comme favorite", Toast.LENGTH_SHORT).show()
        }

        // Lance l'appel API pour récupérer les données météo
        getMeteoFromApi(ville)
    }

    // Gère le clic sur la flèche retour
    override fun onSupportNavigateUp(): Boolean {
        finish()
        return true
    }

    // Renvoie un emoji météo en fonction de la description textuelle
    private fun meteoIcone(description: String?): String {
        return when {
            description == null -> ""
            description.contains("sun", true) || description.contains("clear", true) -> "☀️"
            description.contains("cloud", true) -> "⛅"
            description.contains("rain", true) || description.contains("drizzle", true) -> "🌧️"
            description.contains("thunder", true) -> "🌩️"
            description.contains("snow", true) -> "❄️"
            description.contains("mist", true) || description.contains("fog", true) -> "🌫️"
            else -> "🌈"
        }
    }

    // Appelle l'API météo pour récupérer les prévisions d'une ville
    private fun getMeteoFromApi(ville: String) {
        val call = RetrofitInstance.api.getForecast("1f9cb2ae04d44d21a9780228252305", ville)

        call.enqueue(object : Callback<ForecastResponse> {
            override fun onResponse(call: Call<ForecastResponse>, response: Response<ForecastResponse>) {
                if (response.isSuccessful) {
                    val meteo = response.body()
                    val temp = meteo?.current?.temp_c
                    val condition = meteo?.current?.condition?.text ?: ""
                    val icone = meteo?.current?.condition?.icon

                    // Affiche la température et le texte météo
                    txtTemp.text = "$temp°C - $condition"

                    // Charge l'image météo avec Glide
                    Glide.with(this@MeteoActivity)
                        .load("https:$icone")
                        .into(imgIcone)

                    // Récupère les données horaires pour le matin, l'après-midi et le soir
                    val heures = meteo?.forecast?.forecastday?.get(0)?.hour
                    val matin = heures?.get(8)
                    val aprem = heures?.get(14)
                    val soir = heures?.get(20)

                    // Convertit en ligne de prévision avec les emojis météo
                    val emojiMatin = meteoIcone(matin?.condition?.text)
                    val emojiAprem = meteoIcone(aprem?.condition?.text)
                    val emojiSoir = meteoIcone(soir?.condition?.text)

                    txtJournee.text = "Matin : ${matin?.temp_c}°C $emojiMatin\nAprès-midi : ${aprem?.temp_c}°C $emojiAprem\nSoir : ${soir?.temp_c}°C $emojiSoir"

                    // Crée le résumé pour la semaine avec la température moyenne et un emoji
                    val jours = meteo?.forecast?.forecastday
                    val semaine = jours?.joinToString("\n") { day ->
                        val date = day.date
                        val tempMoyenne = day.day.avgtemp_c
                        val emoji = meteoIcone(day.day.condition.text)
                        "$date : $tempMoyenne°C $emoji"
                    }
                    txtSemaine.text = semaine
                } else {
                    txtTemp.text = "Erreur API"
                }
            }

            override fun onFailure(call: Call<ForecastResponse>, t: Throwable) {
                txtTemp.text = "Erreur : ${t.message}"
            }
        })
    }
}

```
