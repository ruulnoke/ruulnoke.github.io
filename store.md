# Tilanhallinta

Tulisiko sovelluksessa olla store? Yritin pikaperehtyä aiheeseen. Aika paljon löytyi storen hyödyllisyyden kyseenalaistavia näkemyksiä. Yksi toteamus oli, että "jos et tiedä tarvitsetko store, et tarvitse storea." Toisaalta hiljattain julkaistua SignalStorea kehuttiin ja uteliaisuuteni heräsi. 

## NgRx SignalStore

SignalStore on hyvin funktionaalista ohjelmointia ja sen lukemaan ja kirjoittamaan opettelu oli vaikeaa.

Monesti ongelma oli, etten saanut bäkkäriltä sellaista palautusta kuin olisin tarvinnut. Korjasin itsekin välillä niitä bäkkärin puolella. Hyvin usein menin myös sekaisin kaikista sisäkkäisistä funktioista.

Meidän tapauksessamme storessa tehdään lähinnä samat toimenpiteet mitkä tapahtuvat tietokannassa. Turhaa työtä? Toisaalta nyt ei tarvitse hakea koko tietokantaa kuin sovelluksen käynnistettäessä.

Storen metodit:

- projektin luonti, muokkaus ja poisto
- merkinnän luonti, muokkaus ja poisto
- projektin tilan vaihto (valmis, kesken, arkistoitu)
- projektin haku id:n perusteella
- merkinnän haku id:n perusteella
- projektien suodatus tilan perusteella

```typescript
// PROJEKTIN POISTO
    async deleteProject(id: number) {
      await projectService.deleteProject(id);
      // päivitetään tila
      patchState(store, (state) => ({
        projects: state.projects.filter((project) => project.id !== id),
      }));
    },

```

```typescript
// PROJEKTIN HAKU ID:N PERUSTEELLA
    getById(id: number) {
      const project = store.projects().find((project) => project.id === id);
      if (project && project!.updates) {
        // merkintöjen järjestäminen uusin ensin
        project.updates.sort(
          (a, b) =>
            new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
        );
        // merkintösisältöjen järjestäminen järjestysnumeron mukaan
        for (let update of project.updates) {
          update.updateItems.sort((a, b) => a.sequenceNo - b.sequenceNo);
        }
      }
      return project;
    },
```
