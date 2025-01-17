# Tilanhallinta

Tulisiko sovelluksessa olla store? Kokemattomana yritin pikaperehtyä aiheeseen. Aika paljon löytyi storen hyödyllisyyden kyseenalaistavia näkemyksiä. Yksi toteamus oli, että "jos et tiedä tarvitsetko store, et tarvitse storea." Toisaalta hiljattain julkaistua SignalStorea kehuttiin ja uteliaisuuteni heräsi.

## NgRx SignalStore

SignalStore on hyvin funktionaalista ohjelmointia ja sen opettelu oli vaikeaa. Sisäkkäisistä anonyymeista funktioista meni pää pyörälle. Hankalaa oli myös alkuun ymmärtää koko storen ideaa.

Loppujen lopuksi storessamme tehdään pääosin samat toimenpiteet kuin tietokannassakin. Turhaa työtä? Toisaalta nyt ei tarvitse jatkuvasti hakea koko tietokantaa.

### Metodit

- projektin luonti, muokkaus ja poisto
- merkinnän luonti, muokkaus ja poisto
- projektin statuksen vaihto (valmis, kesken, arkistoitu)
- projektin haku id:n perusteella
- merkinnän haku id:n perusteella
- projektien suodatus statuksen perusteella

#### Esimerkkejä

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
// MERKINNÄN MUOKKAUS
    async editProjectUpdate(newupdate: EditedProjectUpdate) {
      const editedUpdate = await projectService.putEditedUpdate(newupdate);
      // päivitetään tila
      patchState(store, (state) => ({
        projects: state.projects.map((project) => {
          if (project.id === newupdate.projectId && project.updates) {
            project.updates = project.updates.map((update) => {
              if (update.id === newupdate.id) {
                // merkinnönjäseniin sijoitetaan muokatut jäsenet
                update.updateItems = editedUpdate.updateItems;
              }
              return update;
            });
          }
          return project;
        }),
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

Storen kanssa operoidessa opin paljon frontin ja bäkkärin välisestä viestinnästä. Monesti ongelmana oli, etten saanut bäkkäriltä sellaista palautusta kuin olisin tarvinnut. Välillä itsekin korjasin niitä bäkkärin puolella.

## Portfolion osiot

- [UI/UX-suunnittelu](design.md)
- [Tietomalli](datamodel.md)
- [Tilanhallinta](store.md)
- [Dynaaminen lomake](forms.md)
- [Product owner](teamwork.md)
- [Etusivu](craftfolio.md)
