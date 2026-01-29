# sql-injection-test
Report: Průnik do databáze pomocí SQL Injection (Union-based)
1. Identifikace zranitelnosti
Prvním krokem bylo ověření, zda je vstupní pole pro vyhledávání uživatelů náchylné k SQL injection. Do vyhledávání byl zadán znak ' (jednoduchá uvozovka), který způsobil chybu nebo změnu chování aplikace. Následně byl použit komentář --, který potvrdil možnost manipulace s dotazem:

Payload: '--

Výsledek: Aplikace vypsala všechny uživatele v databázi, protože došlo k zneplatnění podmínky WHERE v SQL dotazu.

2. Zjištění počtu sloupců (Enumerace)
Aby bylo možné použít příkaz UNION, musí mít podvržený dotaz stejný počet sloupců jako původní dotaz aplikace. Toho bylo dosaženo zkoušením příkazu ORDER BY:

Testování: ' ORDER BY 1--, ' ORDER BY 2--, atd.

Zjištění: Bylo potvrzeno, že aplikace vybírá 2 sloupce (pravděpodobně jméno a pozice).

3. Průzkum struktury databáze (Schema Discovery)
Bylo potřeba zjistit, jaké tabulky se v databázi nacházejí. Jelikož se jednalo o SQLite (časté u těchto úloh), byl dotaz nasměrován na systémovou tabulku sqlite_master:

Payload: ' UNION SELECT name, sql FROM sqlite_master WHERE type='table'--

Výsledek: Výpis názvů tabulek a jejich schémat (příkazů CREATE TABLE). Díky tomu byla identifikována zajímavá tabulka (např. config nebo users).

4. Extrakce dat (Exfiltration)
Po zjištění názvu tabulky a jejích sloupců byl sestaven finální dotaz pro získání citlivých dat (včetně hledaného flagu):

Payload: ' UNION SELECT key, value FROM config--

Výsledek: Aplikace do pole pro jméno a pozici vypsala klíče a hodnoty z tabulky config, kde byl nalezen cílový řetězec (flag).
