# ⚡ Security-Cheatsheets | Red Team Arsenal

**From Reconnaissance to Post-Exploitation**

Этот репозиторий — структурированная коллекция команд, техник и процедур (TTPs), используемых в реальных пентестах и Red Team операциях. Материал основан на опыте, а также на таких известных источниках, как `PayloadsAllTheThings`, `InternalAllTheThings` и `Hacking Articles`.

> **Disclaimer:** Предназначен только для образовательных целей и проведения авторизованных тестирований на проникновение.

---

## 📂 Содержание

| Файл | Описание |
|------|----------|
| **[Ultimate-RedTeam-Sheet.md](./Ultimate-RedTeam-Sheet.md)** | **«Боевой лист»** — все команды в одном файле: шумные и бесшумные техники. |
| **[Active-Directory.md](./Active-Directory.md)** | Атаки на Active Directory: Kerberos (AS-REP, Kerberoast), ACL Abuse, GPO, NTLM Relay, DCSync. |
| **[Web-Application.md](./Web-Application.md)** | Веб-уязвимости: SQLi, XSS, SSTI, SSRF, XXE, JWT, GraphQL, массовые атаки. |
| **[Linux-PrivEsc.md](./Linux-PrivEsc.md)** | Повышение привилегий в Linux: SUID, Capabilities, Cron, Docker Breakout. |
| **[Windows-PrivEsc.md](./Windows-PrivEsc.md)** | Повышение привилегий в Windows: JuicyPotato, Token Impersonation, UAC Bypass. |
| **[C2-Frameworks.md](./C2-Frameworks.md)** | Работа с C2: Sliver, Viper, Havoc — создание стейджеров, обход AV. |
| **[Hardware-IoT.md](./Hardware-IoT.md)** | Аппаратная безопасность: UART, SPI, JTAG, анализ прошивок. |

---

## 🚀 Быстрый старт

```bash
git clone https://github.com/Artur0103/Security-Cheatsheets.git
cd Security-Cheatsheets
cat Ultimate-RedTeam-Sheet.md
