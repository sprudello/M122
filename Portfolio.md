# Portfolioeintrag

Julius Valentin Burlet

# Einleitung
Das Thema für diesen Portfolioeintrag ist, ein Skript mit PowerShell zu programmieren. Mein Skript kann durch das Lesen vom Netzwerknamen verschiedene „Profile“ starten. Im Falle dieses Programms startet es, wenn man im BBB Netzwerk ist, https://moodle.bbbaden.ch und öffnet den Hauptordner der BBB, wo alle Aufträge drin sind.

# Sinn und Zweck dieses Programmes
Dieses Skript automatisiert den Anfang des Tages. Man kommt in die Schule, loggt sich ein und man hat direkt den Ordner für die Schule und die wichtigste Webseite offen für die Schule. Es ist also nur ein Tool, welches den Arbeitsablauf verbessern soll.
Ist man in einem anderen oder in keinem Netzwerk wird man gefragt, ob man in der Schule ist, gibt man nichts ein, schliesst es das Skript.

# Beschreibung des Themas
Im Modul geht es um Skriptsprachen, vorwiegend PowerShell, welches primär dafür da ist, Dinge zu automatisieren.

# Beschreibung

## Part 1 | Speicherung und Lesen der config.json File.

Damit der Pfad für die Hauptordner der Schulen gespeichert werden können, muss man diese Dateipfade in einer Datei speichern. In meinem Fall war dies eine .json File.

### Schreiben der Datei

Der Code dafür sieht wie folgt aus:
```ps1
if (-not (Test-Path -Path $ConfigFilePath)) {
            # Prompt the user for Kanti and BBB main folder paths
            $kantiPath = Read-Host "What is your Kanti main folder: [path]"
            $bbbPath = Read-Host "What is your BBB main Folder: [path]"

            # Check if the provided paths are valid
            if (-not (Test-Path -Path $kantiPath) -or (-not (Test-Path -Path $bbbPath))) {
                Throw
            }
        }
        # Create the configuration content
        $configContent = @{
            kantifolder = $kantiPath
            bbbfolder   = $bbbPath
        }

        # Convert the configuration content to JSON and save it to the configuration file
        $configContent | ConvertTo-Json | Out-File -FilePath $ConfigFilePath -Encoding UTF8
        Write-Host "config.json file created successfully."
        break
```
Zuerst wird überprüft, ob diese Datei bereits existiert, falls nicht, wird diese kreiert. Dafür benötigt man erst die Dateipfade für die Ordner, welche eingegeben werden können und auf ihre Validität überprüft werden.
Danach werden die Pfade in `$configContent` gespeichert und umgewandelt, damit man die Pfade in ein .json File einfügen kann, ohne reading Errors zu bekommen. 

### Lesen der Datei

```ps1
# Read the configuration file content
$jsonContent = Get-Content -Raw -Path $ConfigFilePath | ConvertFrom-Json

# Retrieve the Kanti and BBB folder paths from the configuration
$kantiPath = $jsonContent.kantifolder
$bbbPath = $jsonContent.bbbfolder
```
Obwohl das Schreiben der Datei bisschen komplexer ist, ist das Lesen umso einfacher. Die Datei wird grundsätzlich einfach gelesen und die beiden Pfade werden in den Variablen `$kantiPath` und `$bbbPath` gespeichert.

 ## Part 2 | Überprüfung des Netzwerks und Start der Profile

 ```ps1
# Launch URLs and folders based on the network profile
$networkProfile = Get-NetConnectionProfile

if ($networkProfile.Name -eq "bbbaden.local") {
    Start https://moodle.bbbaden.ch
    Invoke-Item -Path $bbbPath
}
elseif ($networkProfile.Name -eq "kantibaden.local") {
    Start https://www.schul-netz.com/baden/index.php
    Invoke-Item -Path $kantiPath
 ```
Das ist einer der einfachsten Teile des Skripts. Es überprüft durch das NetConnectionProfile und den Namen des Netzwerks in welchem Netzwerk man ist. Heisst, ist man im BBBaden Netzwerk ist das if Statement true und es startet automatisch das BBB Profil.

## Part 3 | Kein Netz?

Falls man kein Netz oder weder im Kanti noch im BBB Netzwerk ist, kommt man zu einer manuellen Eingabe.

```ps1
else {
    while ($true) {
        try {
            $setup = Read-Host "Where are you? bbb or kanti"
            if ($setup -eq "bbb") {
                Start https://moodle.bbbaden.ch
                Invoke-Item -Path $bbbPath
                break
            }
            elseif ($setup -eq "kanti") {
                Start https://www.schul-netz.com/baden/index.php
                Invoke-Item -Path $kantiPath
                break
            }
            elseif ($setup -eq "") {
                Exit
            }
            else {
                Throw 
            }
        }
```
Hiermit kann man noch manuell zwischen `bbb` und `kanti` entscheiden. Wenn man nichts schreibt, schliesst sich das Programm direkt, ohne etwas zu starten.

# Reflexion

## Was ist gut gelaufen
Die Idee hatte ich rasch, ein Konzept im Kopf auch. Bei der Umsetzung ging es gut voran.

## Was ist nicht so gut gelaufen
Ich hatte Schwierigkeiten mit dem .json File, da man Dateipfade nicht ohne Weiteres drin speichern kann.

## Folgerung
Hätte ich zuerst nach .json gesucht, hätte ich wahrscheinlich diesen Fehler, noch vor dem Entstehen verhindern können. Ich sollte, wenn ich mit etwas Neuem arbeite, zuerst danach googeln und schauen, ob und welche Regeln dieser Dinge haben.
