# Mantrego - wydania

To repozytorium zawiera wyłącznie gotowe pliki do pobrania aplikacji Mantrego: instalator Windows, aplikację Android (APK) oraz podpisane manifesty aktualizacji. Kod źródłowy aplikacji nie jest tutaj publikowany.

Strona produktu: https://mantrego.com

## Pliki w każdym wydaniu

| Plik | Do czego służy |
|---|---|
| `Mantrego-Setup-<wersja>.exe` | Instalator Windows (64-bit). |
| `Mantrego-<wersja>.apk` | Aplikacja Android. |
| `latest.json` | Podpisany manifest aktualizacji Windows. Aplikacja sprawdza go automatycznie. |
| `latest-android.json` | Podpisany manifest aktualizacji Android. |
| `SHA256SUMS.txt` | Sumy SHA-256 wszystkich plików wydania. |

Najnowsze wydanie: https://github.com/kontaktjakubjagiello/mantrego-releases/releases/latest

## Jak działają aktualizacje

Zainstalowana aplikacja Windows po uruchomieniu i co 6 godzin pobiera `latest.json` z najnowszego wydania. Najpierw sprawdza podpis Ed25519 manifestu kluczem publicznym wbudowanym w aplikację, dopiero potem pobiera instalator i porównuje jego rozmiar oraz sumę SHA-512 z podpisanym manifestem. Instalacja rusza tylko po kliknięciu "Aktualizuj".

Aplikacja Android pokazuje informację o nowej wersji i link do pliku APK. Instalację zatwierdzasz w systemie Android.

## Jak samodzielnie sprawdzić pliki

Suma pliku (Windows PowerShell):

```powershell
Get-FileHash .\Mantrego-Setup-<wersja>.exe -Algorithm SHA256
```

Wynik porównaj z wierszem w `SHA256SUMS.txt` i w opisie wydania.

Podpis manifestu (Node.js 20 lub nowszy). Zapisz klucz publiczny wydawcy jako `update-key.pem`:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAZAQmoODMxkehgqnTcD3KxMwDvbG0lPjdpYiSjQaPntQ=
-----END PUBLIC KEY-----
```

Następnie uruchom w folderze z pobranym `latest.json`:

```
node -e "const c=require('crypto'),f=require('fs');const e=JSON.parse(f.readFileSync('latest.json'));console.log(c.verify(null,Buffer.from(JSON.stringify(e.manifest)),f.readFileSync('update-key.pem'),Buffer.from(e.signature,'base64'))?'Podpis poprawny':'PODPIS NIEPOPRAWNY')"
```

Podpis obejmuje cały obiekt `manifest`, w tym wersję, nazwę pliku, rozmiar i sumę SHA-512 instalatora (dla Androida SHA-256 pliku APK). Ten sam test działa dla `latest-android.json`.

Instalator Windows nie ma jeszcze komercyjnego certyfikatu Authenticode, dlatego przy pierwszej instalacji Windows może pokazać ostrzeżenie SmartScreen.
