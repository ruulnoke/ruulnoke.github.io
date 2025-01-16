# Tietomalli

## Tietokantasuunnitelma

Loin mallin SQL-komentoina ja niistä Workbenchin reverse engineer -työkalulla.

```sql
CREATE DATABASE IF NOT EXISTS craftfolio;
USE craftfolio;

CREATE TABLE Kayttaja (
tunnus VARCHAR(255) CHARACTER SET ascii NOT NULL,
etunimi VARCHAR(60) NOT NULL,
sukunimi VARCHAR(60) NOT NULL,
premium BOOLEAN DEFAULT false NOT NULL,
PRIMARY KEY (tunnus)
) ENGINE=InnoDB;

CREATE TABLE Projekti (
id INT AUTO_INCREMENT,
nimi VARCHAR(24) NOT NULL,
kuva VARCHAR(8192),
kuvaus VARCHAR(1000),
tekniikat VARCHAR(1000),
materiaalit VARCHAR(1000),
ong_ja_oiv VARCHAR(2000),
luotu TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
paivitetty TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
suosikki BOOLEAN DEFAULT false NOT NULL,
tila ENUM ('kesken', 'valmis', 'arkistoitu') NOT NULL,
kayttaja_tunnus VARCHAR(255) CHARACTER SET ascii,
PRIMARY KEY (id),
FOREIGN KEY (kayttaja_tunnus) REFERENCES Kayttaja(tunnus)
) ENGINE=InnoDB;

CREATE TABLE Paivitys (
id INT AUTO_INCREMENT,
otsikko VARCHAR(34),
luotu TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
projekti_id INT,
PRIMARY KEY (id),
FOREIGN KEY (projekti_id) REFERENCES Projekti(id)
) ENGINE=InnoDB;

CREATE TABLE PaivityksenKohde (
id INT AUTO_INCREMENT,
kuva VARCHAR(8192),
teksti VARCHAR(2000),
constraint vain_yksi_paivityssisalto
        check (        (kuva is null or teksti is null)
               and not (kuva is null and teksti is null) ),
jarjestysnro INT NOT NULL,
paivitys_ID INT,
PRIMARY KEY (id),
FOREIGN KEY (paivitys_id) REFERENCES Paivitys(id)
) ENGINE=InnoDB;

CREATE TABLE Lista (
id INT AUTO_INCREMENT,
nimi VARCHAR(24) NOT NULL,
luotu TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
paivitetty TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
suosikki BOOLEAN DEFAULT false NOT NULL,
kayttaja_tunnus VARCHAR(255) CHARACTER SET ascii,
PRIMARY KEY (id),
FOREIGN KEY (kayttaja_tunnus) REFERENCES Kayttaja(tunnus)
) ENGINE=InnoDB;

CREATE TABLE ListanKohde (
id INT AUTO_INCREMENT,
kuva VARCHAR(8192),
teksti VARCHAR(500),
constraint vain_yksi_listasisalto
        check (        (kuva is null or teksti is null)
               and not (kuva is null and teksti is null) ),
jarjestysnro INT NOT NULL,
lista_id INT,
PRIMARY KEY (id),
FOREIGN KEY (lista_id) REFERENCES Lista(id)
) ENGINE=InnoDB;



```

![tietokantakaavio](images\craftfolio-tietokanta.PNG)

### Kieli- ja nimeämisongelmat

Johtuen varmaankin käymistäni kursseista, aloin automaattisesti luoda tietokantaa suomenkielisenä.

## Rajapinnat

Projektiin liittyviä rajapintoja tarvittiinkin enemmän kuin ensin kuvittelin.

```typescript
// -----MERKINTÄ----

// käyttöliittymässä luodun merkintäsisällön rajapinta
export interface BaseProjectUpdateItem {
  text: string;
  image: string;
  contentType: 'text' | 'image';
  sequenceNo: number;
  projectUpdateId: number;
}

// tietokantaan tallennetun merkintäsisällön rajapinta
export interface ProjectUpdateItem extends BaseProjectUpdateItem {
  id: number;
}

// käyttöliittymässä luodun merkinnän rajapinta
export interface BaseProjectUpdate {
  title?: string;
  updateItems: ProjectUpdateItem[];
  projectId: number;
}

// tietokantaan tallennetun merkinnän rajapinta
export interface ProjectUpdate extends BaseProjectUpdate {
  id: number;
  createdAt: string;
}

// rajapinta lähetykselle muokatessa merkintöä
export interface EditedProjectUpdate extends BaseProjectUpdate {
  id: number;
  deletedItems: number[];
}

// rajapinta palautukselle luodessa tai muokatessa merkintöjä
export interface ProjectUpdateResponse {
  0: ProjectUpdate;
  1: ProjectUpdateItem[];
}

// -----PROJEKTI----

// käyttöliittymässä luodun projektin rajapinta
export interface BaseProject {
  name: string;
  image: string;
  description?: string;
  techniques?: string;
  materials?: string;
  insights?: string;
  favorite: Boolean;
  updates?: ProjectUpdate[];
}
// käyttöliittymässä muokatun projektin rajapinta
export interface EditedProject extends BaseProject {
  id: number;
}

// tietokantaan tallennetun projektin rajapinta
export interface Project extends BaseProject {
  id: number;
  createdAt: string;
  updatedAt: string;
  status: 'started' | 'completed' | 'archived';
}

// rajapinta projektistatuksen muokkaukseen
export interface ProjectStatus {
  id: number;
  status: 'started' | 'completed' | 'archived';
}
```

## Portfolion osiot

- [Product owner](teamwork.md)
- [UI/UX-suunnittelu](design.md)
- [Tietomalli](datamodel.md)
- [Tilanhallinta](store.md)
- [Komponentit](forms.md)
