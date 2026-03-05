# Kontekst Projektu: Photo Events (Calendar MCP)

Ten plik służy jako dokumentacja kontekstowa dla agentów AI pracujących z tym repozytorium. Opisuje on cel projektu, jego architekturę oraz sposób interakcji z kalendarzem Google poprzez zaimplementowany serwer MCP.

## Czym jest to repozytorium?

Repozytorium `photo-events` zawiera implementację serwera **MCP (Model Context Protocol)**, który działa jako most między asystentem AI (Gemini, Claude) a API Kalendarza Google.

### Kluczowe funkcjonalności:
*   **Bezpieczeństwo:** Serwer działa lokalnie (`127.0.0.1`), wykorzystuje OAuth 2.0 i przechowuje tokeny lokalnie.
*   **Zarządzanie Wydarzeniami:** Tworzenie, edycja, usuwanie i wyszukiwanie wydarzeń.
*   **Zarządzanie Kalendarzami:** Listowanie i tworzenie nowych kalendarzy.
*   **Zaawansowane funkcje:** Analiza zajętości, sprawdzanie statusu gości oraz **obsługa niestandardowych przypomnień** (funkcja dodana 2026-02-01).

## Konfiguracja Środowiska
*   **Lokalizacja:** Projekt znajduje się w `/home/xai/DEV/photo-events/`.
*   **Uruchamianie:** Serwer jest uruchamiany automatycznie przez klienta MCP dzięki konfiguracji w `.gemini/settings.json`. Wykorzystuje wirtualne środowisko `.venv`.
*   **Pliki kluczowe:**
    *   `run_server.py`: Punkt wejścia serwera.
    *   `src/mcp_bridge.py`: Definicje narzędzi (tools) wystawianych dla AI.
    *   `.gcp-saved-tokens.json`: Zapisane tokeny uwierzytelniające.

---

## Jak dodawać wydarzenia (Events)

Aby dodać wydarzenie do kalendarza, asystent powinien skorzystać z narzędzia `create_event` (lub `calendar-mcp__create_event`).

### Parametry narzędzia `create_event`:

| Parametr | Typ | Wymagany | Opis |
| :--- | :--- | :---: | :--- |
| `calendar_id` | `str` | TAK | ID kalendarza (np. `primary` lub email grupy np. `...@group.calendar.google.com`). |
| `summary` | `str` | TAK | Tytuł wydarzenia. |
| `start_time` | `str` | TAK | Data i czas rozpoczęcia w formacie ISO (np. `2026-03-23T08:00:00`). |
| `end_time` | `str` | TAK | Data i czas zakończenia w formacie ISO. |
| `description` | `str` | NIE | Szczegółowy opis wydarzenia. |
| `location` | `str` | NIE | Miejsce wydarzenia. |
| `attendee_emails`| `List[str]`| NIE | Lista emaili gości do zaproszenia. |
| **`reminder_minutes`**| `List[int]`| NIE | Lista minut przed wydarzeniem dla alertów (np. `[1440, 60]`). |
| **`reminder_methods`**| `List[str]`| NIE | Metody alertów (musi pasować długością do minut, domyślnie `popup`). Opcje: `popup`, `email`. |

### Przykład użycia (Tool Call)

Aby dodać wydarzenie z przypomnieniami na 7 dni i 1 dzień przed:

```json
{
  "name": "calendar-mcp__create_event",
  "args": {
    "calendar_id": "9pvcn5nlplvejrc9npeco7u6a4@group.calendar.google.com",
    "summary": "Sesja Zdjęciowa DWC",
    "start_time": "2026-03-23T08:00:00",
    "end_time": "2026-03-23T22:00:00",
    "location": "Nowohuckie Centrum Kultury, Kraków",
    "description": "Organizator: L'Art de la Danse",
    "reminder_minutes": [10080, 1440],
    "reminder_methods": ["popup", "popup"]
  }
}
```

### Szybkie dodawanie (Quick Add)
Dla prostych wydarzeń bez zaawansowanych ustawień można użyć `quick_add_event`:
*   **Argument:** `text` (np. "Spotkanie z klientem jutro o 10am").

## Aktualizacja wydarzeń
Użyj narzędzia `update_event`. Obsługuje ono te same parametry co tworzenie (w tym `reminder_minutes`), pozwalając na modyfikację istniejących wpisów. Wymaga podania `event_id`.
