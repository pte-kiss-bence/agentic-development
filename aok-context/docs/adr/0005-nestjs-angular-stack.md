---
status: accepted
---

# Állandó stack: NestJS + Angular (minden projekten)

Minden projekten a **NestJS (backend) + Angular (frontend)** kombóval haladunk — ez a csapat állandó, alapértelmezett stackje, nem projektenként újratárgyalt döntés. Ehhez a projekthez az adatbázis **PostgreSQL**.

## Considered Options

- *ASP.NET Core (.NET) + SQL Server* (az openspec eredeti D1 ajánlása) — **elvetve**. Nem a csapat stackje; kemény határidőnél a meglévő tudásra építünk, és a „forrás-DB valószínűleg SQL Server" érv nem indokol .NET-et (az egyszeri ETL bármely stackből olvashat SQL Serverből).

## Consequences

- Nem tervezünk .NET-tel; a jövőben se javasolja senki újra. Az Entra/M365 integráció NestJS/Angular oldalon (MSAL + JWT-validáció) történik.
