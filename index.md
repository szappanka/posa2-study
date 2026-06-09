---
layout: default
title: Kezdőlap
---

# POSA2 Study Guide

Tanulási jegyzetek a *Pattern-Oriented Software Architecture, Volume 2* alapján.
A **Service Access & Configuration** kategória patternjeit dolgozzuk fel.

<div class="research-card-wrap">
  <a class="research-card" href="{{ '/docs/kutatasi-patternek' | relative_url }}">
    <span class="research-card-icon">🔬</span>
    <span class="research-card-body">
      <strong>POSA2 a saját kutatásomban</strong>
      <span>Hogyan jelennek meg ezek a patternek a szemantikus környezetmodellezésben és a 2D point cloud regisztrációban — Unity és MRTK alapú rendszerekben.</span>
    </span>
    <span class="research-card-arrow" aria-hidden="true">→</span>
  </a>
</div>

## Témakörök

### [Service Access & Configuration]({{ '/docs/01-service-access-overview' | relative_url }})

A kategória négy patternje és az általuk megoldott tervezési kihívások.

**Patternek:**

- [Wrapper Facade]({{ '/docs/02-wrapper-facade' | relative_url }})
  A csúnya C API-k OOP becsomagolása, hordozhatóság, biztonság, karbantarthatóság.
- [Component Configurator]({{ '/docs/03-component-configurator' | relative_url }})
  Komponensek be- és kikapcsolása futás közben, az alkalmazás újraindítása nélkül.
- [Interceptor]({{ '/docs/04-interceptor' | relative_url }})
  Kiterjeszthetőség: saját kód beépítése a keretrendszer feldolgozási folyamatába.
- [Extension Interface]({{ '/docs/05-extension-interface' | relative_url }})
  Több interfész egy komponensen, hogy bővíthető legyen a kliensek érintése nélkül.

---

*A quiz-app később kerül ide.*
