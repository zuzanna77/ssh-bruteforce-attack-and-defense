# Atak brute-force na usługę SSH – atak i zabezpieczenie

Projekt zaliczeniowy z przedmiotu **Bezpieczeństwo sieci komputerowych**.
Praca pokazuje w kontrolowanym środowisku laboratoryjnym, jak wygląda zautomatyzowany atak słownikowy na usługę SSH, jak jest widoczny w logach serwera oraz jak można się przed nim zabezpieczyć.

> **Uwaga:** wszystkie działania wykonano wyłącznie w odizolowanym, wirtualnym środowisku laboratoryjnym, na własnych maszynach. Repozytorium ma charakter edukacyjny. Nie wolno wykorzystywać opisanych technik wobec systemów, do których nie masz wyraźnej zgody właściciela.

## Dokumentacja

Pełny opis projektu (teoria, środowisko, przebieg ataku, zabezpieczenia, wnioski) znajduje się w pliku:

📄 [`Atak_brute-force_na_usluge_SSH.pdf`](./Atak_brute-force_na_usluge_SSH.pdf)

## Zakres projektu

1. **Wstęp teoretyczny** – protokół SSH, różnica między czystym brute-force a atakiem słownikowym.
2. **Środowisko laboratoryjne** – topologia w GNS3 + maszyny wirtualne w VirtualBox.
3. **Atak** – rekonesans, atak słownikowy na konto z słabym hasłem, analiza logów, weryfikacja dostępu.
4. **Obrona** – Fail2Ban, utwardzenie `sshd_config`, uwierzytelnianie kluczami.
5. **Weryfikacja** – ponowny test po wdrożeniu zabezpieczeń i porównanie stanu przed/po.
6. **Wnioski**.

## Środowisko

| Urządzenie    | Adres IP       | Rola                      |
|---------------|----------------|---------------------------|
| Kali Linux    | 192.168.1.10/24 | Atakujący                |
| Ubuntu Server | 192.168.1.20/24 | Ofiara (serwer OpenSSH)  |
| Router Cisco 3725 | 192.168.1.1/24 | Brama domyślna        |

- **GNS3** – emulacja infrastruktury sieciowej (switch, router Cisco IOS),
- **Oracle VirtualBox** – maszyny z prawdziwymi systemami (Kali Linux, Ubuntu Server),
- cała sieć jest izolowana od sieci zewnętrznych.

## Wykorzystane narzędzia

| Kategoria        | Narzędzie                                   |
|------------------|---------------------------------------------|
| Rekonesans       | Nmap                                        |
| Atak             | Hydra, słownik `rockyou.txt`                |
| Analiza          | Wireshark, logi systemowe (`auth.log`)      |
| Obrona           | Fail2Ban, OpenSSH (`sshd_config`, klucze RSA) |

## Przebieg i wyniki

**Faza ataku**
- Nmap wykrył otwarty port 22/tcp z usługą OpenSSH 9.6p1 (Ubuntu).
- Atak słownikowy na konto testowe z celowo słabym hasłem zakończył się powodzeniem w ciągu kilkudziesięciu prób.
- W `auth.log` widoczne były setki wpisów `Failed password` w krótkim czasie – charakterystyczny ślad automatu.

**Faza obrony**

| Parametr            | Przed             | Po                          |
|---------------------|-------------------|-----------------------------|
| Port SSH            | 22                | 2222                        |
| Logowanie hasłem    | włączone          | wyłączone                   |
| Limit prób          | brak              | `MaxAuthTries 3` + Fail2Ban (3 próby, ban 10 min) |
| Logowanie roota     | `prohibit-password` | `no`                      |
| Uwierzytelnianie    | hasło             | klucz RSA 4096 bit          |

**Wniosek:** najlepsze rezultaty daje podejście warstwowe (*defense in depth*): Fail2Ban reaguje na zachowanie atakującego, zmiana portu ogranicza szum od botów, a uwierzytelnianie kluczem eliminuje sam wektor zgadywania haseł.

## Struktura repozytorium

```
.
├── README.md
└── Atak_brute-force_na_usluge_SSH.pdf
```

## Autorki

- Amelia Skoczylas
- Zuzanna Lech
