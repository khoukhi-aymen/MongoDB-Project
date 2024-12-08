artie II : Création des TablesSpaces et des utilisateurs*/
/*--- création des tablespace-------------*/
create tablespace SQL33_TBS  datafile 'C:\tbs\SQL33_TBS.dat' size 100M autoextend on online;
create temporary tablespace SQL33_TempTBS tempfile 'C:\tbs\SQL33_TempTBS.dat' size 100M autoextend on;
--- création de l'utilisateur------------
create user SQL3 identified by psw default tablespace SQL33_TBS temporary tablespace SQL33_TempTBS;
---- accorder les touts les droits----------
grant all privileges to SQL3 ;
--- connexion avec l'utilisateur SQL3  
connect SQL3/psw

/*Partie III : Langage de définition de données*/



/*--- Définition de tous les types incomplet -------------*/




create type T_Sportif;/
create type T_Sport;/
create type T_Arbitrer;/
create type T_Jouer;/
create type T_Entrainer;/
create type T_ville;/
create type T_Gymnass;/
create type T_Seance;/
/*--- Définition de tous les types défini par utilisateur -------------*/
                     --//table Sportif----------
create type T_set_ref_Arbitrer as table of ref T_Arbitrer;/
create type T_set_ref_Jouer as table of ref T_Jouer;/
create type T_set_ref_Seance as table of ref T_Seance;/
create type T_set_ref_Sportif as table of ref T_Sportif;/
create type T_set_ref_Entrainer as table of ref T_Entrainer;/
create type T_set_ref_sport as table of ref T_Sport;/
create type T_set_ref_Gymnass as table of ref T_Gymnass;/
   --//Mise à jour de type icomplet T_Sportif :
create or replace type T_Sportif as object ( 
  IDSPORTIF number,
  Nom varchar2(50),
  prenom varchar2(50),
  sexe varchar2(50),
  Age number,
  sportif_arbitrer  T_set_ref_Arbitrer,
  sportif_jouer     T_set_ref_Jouer,
  est_conseile      T_set_ref_Sportif,
  sportif_seance    T_set_ref_Seance,
  sportif_Entrainer T_set_ref_Entrainer
  );
 --nb: j'ai oublie le role Sportif_sport donc je vais l'ajouter 
 alter type T_Sportif add attribute sportif_Sport T_set_ref_Sport CASCADE;

 --//-------------------------------------------------------------// 
                       --//table Sport----------
--//les types collection existent déja 

                --//Mise à jour de type icomplet T_Sport :
create or replace type T_Sport as object ( 
  IDSPORT number,
  Libelle varchar2(50),
  sport_arbitrer  ref T_Arbitrer,
  sport_jouer     T_set_ref_Jouer,
  sport_seance    T_set_ref_Seance,
  sport_Entrainer T_set_ref_Entrainer
  );/
 --nb: j'ai oublie le role Sport_sportif donc je vais l'ajouter 
 alter type T_Sport add attribute sport_Sportif T_set_ref_sportif CASCADE;
  --nb: j'ai oublie le role Sport_gymnass donc je vais l'ajouter 
alter type T_Sport add attribute Sport_gymnass T_set_ref_Gymnass cascade;
 
 --//-------------------------------------------------------------// 
                       --//table Arbitrer----------

                --//Mise à jour de type icomplet T_Sport :
create or replace type T_Arbitrer as object( 
  arbitrer_sportif      T_set_ref_Sportif,
  arbitrer_sport        T_set_ref_Sport
  );/
--//-------------------------------------------------------------// 
                       --//table Jouer----------
--//les types collection existent déja
                --//Mise à jour de type icomplet T_Sport :
				
create or replace type T_Jouer as object( 
  jouer_sportif      T_set_ref_Sportif,
  jouer_sport        T_set_ref_Sport
  );/
  
--//-------------------------------------------------------------// 
                       --//table Entrainer----------
--//les types collection existent déja
                --//Mise à jour de type icomplet T_Entrainer :
				
create or replace type T_Entrainer as object( 
  entrainer_sportif      ref T_Sportif,
  entrainer_sport        T_set_ref_Sport
  );/
--//-------------------------------------------------------------// 
                       --//table Ville----------
                --//Mise à jour de type icomplet T_Ville :
				
create or replace type T_ville as object( 
  ville                  varchar2(50),
  ville_gymnass          T_set_ref_Gymnass
  );/
  
--//-------------------------------------------------------------// 
                       --//table Gymnass----------
--//les types collection existent déja
                --//Mise à jour de type icomplet T_Gymnass :
				
create or replace type T_Gymnass as object( 
  IDGYMNASE              number,
  NOMGYMNASE             varchar2(50),
  ADRESSE                varchar2(50),
  SURFACE                FLOAT(20),
  gymnass_seance         T_set_ref_Seance,
  gymnass_ville          ref T_ville
  );/
--//-------------------------------------------------------------// 
                       --//table Seance----------
                --//Mise à jour de type icomplet T_Seance :
				
create or replace type T_Seance as object( 
  Jour                   varchar2(50),
  Horaire                FLOAT(20),
  Duree                  number,
  Seance_Sportif         ref T_Sportif,
  Seance_gymnass         ref T_Gymnass, 
  Seance_Sport           ref T_Sport
  );/
  
  
  
  
/*--- Définition de tous les méthodes ---------------------------*/  
 
 
 
 
 
 --//-------------------------------------------------------------// 				   
/*--- méthode de Calcule pour chaque sportif, le nombre des sports entrainés.-------------*/
   //----------la signature de la méthode ---------------
alter type T_Sportif add member function nbr_sports_entraines return number cascade;
 --//----------le corps de la méthode ---------------
create or replace type body T_Sportif as 
  member function nbr_sports_entraines return number is -- Ajout du "is" ici
    nbr_sports_entr number;
  begin
    select count(deref(value(T1))) into nbr_sports_entr 
    from sportif s, table(s.sportif_Sport) T1
    where s.IDSPORTIF = self.IDSPORTIF;
    return nbr_sports_entr; 
  end nbr_sports_entraines;
end;
/ 
 --//-------------------------------------------------------------// 				   
/*--- méthode de Calcule pour chaque sport, le nombre de gymnass.-------------*/
   --//----------la signature de la méthode ---------------
alter type T_Sport add member function nbr_gymnass_sport return number cascade;
 --//----------le corps de la méthode ---------------
create or replace type body T_Sport as 
  member function nbr_gymnass_sport return number is -- Ajout du "is" ici
    nbr_gym_sport number;
  begin
    select count(deref(value(t2))) into nbr_gym_sport 
    from sport s, table(s.sport_gymnass) t2
    where s.IDSPORT = self.IDSPORT;
    return nbr_gym_sport; 
  end nbr_gymnass_sport;
end;
/ 

 --//-------------------------------------------------------------// 				   
/*--- méthode de Calcule pour chaque ville, la superficie moy de gymnass.-------------*/
                  --//----------la signature de la méthode ---------------

alter type T_ville add member function calcul_sup_moy_gymnass return float cascade;
                  --//----------   le corps de la méthode  ---------------
create or replace type body T_ville as 
  member function calcul_sup_moy_gymnass return float is -- Ajout du "is" ici
    sup_moy_gym float;
  begin
    select AVG(deref(value(t1)).surface) into sup_moy_gym 
    from ville v, table(v.ville_gymnass) t1 
    where v.ville = self.ville;
    return sup_moy_gym; 
  end calcul_sup_moy_gymnass;
end;
/ 

/*--- Définition de tous les tables nécessaire à la base de données ---------------------------*/


              --//table Sportif----------
--//nb : jai oubliée l'attribut Conseille donc je vais l'ajouter a cet table
alter type T_Sportif add attribute Conseille ref T_Sportif cascade; 


create table Sportif of T_Sportif(primary key(IDSPORTIF),foreign key(Conseille) references Sportif)
nested table sportif_arbitrer store as table_sportif_arbitrer,
nested table sportif_jouer store as table_sportif_jouer,
nested table est_conseile store as table_est_conseile,
nested table sportif_seance store as table_sportif_seance,
nested table sportif_Entrainer store as table_sportif_Entrainer,
nested table sportif_sport store as table_sportif_sport;


               --//table Arbitrer----------
create table Arbitrer of T_Arbitrer
nested table arbitrer_sportif store as table_arbitrer_sportif,
nested table arbitrer_sport store as table_arbitrer_sport;



               --//table Jouer----------
			   
			   
create table Jouer of T_Jouer
nested table jouer_sport store as table_jouer_sport,
nested table jouer_sportif store as table_jouer_sportif;

              --//table Entrainer----------
			   
			   
create table Entrainer of T_Entrainer(foreign key(entrainer_sportif) references Sportif)
nested table entrainer_sport store as table_entrainer_sport;	

              --//table ville----------

			  
create table Ville of T_ville
nested table ville_gymnass store as table_ville_gymnass;


               --//table gymnass----------
--//nb :on va ajouter l'attribut gymnass_sport car j'ai fait une mise a jour au niveau de diagramme de classe lors de la création de la méthode nbr_gym_sport
alter type T_Gymnass add attribute gymnass_sport T_set_ref_Sport cascade; 


create table Gymnass of T_Gymnass(primary key(IDGYMNASE),foreign key(gymnass_ville) references Ville)
nested table gymnass_seance store as table_gymnass_seance,
nested table gymnass_sport store as table_gymnass_sport;


               --//table Sport----------
  
CREATE TABLE Sport OF T_Sport (
    PRIMARY KEY (IDSPORT),
    FOREIGN KEY (SPORT_ARBITRER) REFERENCES Arbitrer
)
NESTED TABLE sport_jouer STORE AS table_sport_jouer,
NESTED TABLE sport_seance STORE AS table_sport_seance,
NESTED TABLE sport_Entrainer STORE AS table_sport_Entrainer,
NESTED TABLE SPORT_GYMNASS STORE AS table_sport_gymnass,
nested table sport_sportif store as table_sport_sportif;


             --//table seance----------
			 
			 
 
CREATE TABLE Seance of T_Seance(
    FOREIGN KEY (Seance_Sportif) REFERENCES Sportif,
    FOREIGN KEY (Seance_Sport) REFERENCES sport,
    FOREIGN KEY (Seance_gymnass) REFERENCES gymnass,
    CHECK (Jour IN ('SAMEDI', 'DIMANCHE', 'LUNDI', 'MARDI', 'MERCREDI', 'JEUDI', 'VENDREDI'))
);


/*Partie III : Langage de manipulation de données*/

--//------------------------les insertions----------------------------------------------------------

                  --//---------------------------->table Sportif <-----------------------------------
					
					
					
  --//le sportif BOUTAHAR Abderahim conseille le sportif de IDSPORT = 2 et IDSPORTIF 3 et IDSPORTIF 4 et il na pas de conseillé
INSERT INTO Sportif VALUES(
    1, 'BOUTAHAR', 'Abderahim', 'M', 30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 2),
            (select ref(s) from Sportif s where s.IDSPORTIF = 3),
            (select ref(s) from Sportif s where s.IDSPORTIF = 4)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);

--//le sportif BOUROUBI Anis conseille le sportif de IDSPORT = 5 et IDSPORTIF 6 et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    2, 'BOUROUBI', 'Anis', 'M', 28,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 5),
            (select ref(s) from Sportif s where s.IDSPORTIF = 6)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);
--//le sportif BOUZIDI Amel conseille le sportif de IDSPORT = 7 et IDSPORTIF 8 et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    3, 'BOUZIDI', 'Amel', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 7),
            (select ref(s) from Sportif s where s.IDSPORTIF = 8)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);
--//le sportif LACHEMI Bouzid conseille le sportif de IDSPORT = 7 et IDSPORTIF 8 et IDSPORT = 9 et IDSPORTIF 10 et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    4, 'LACHEMI', 'Bouzid', 'M',32,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 7),
            (select ref(s) from Sportif s where s.IDSPORTIF = 8),
			(select ref(s) from Sportif s where s.IDSPORTIF = 9),
			(select ref(s) from Sportif s where s.IDSPORTIF = 10)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);
--//le sportif AAKOUB Linda conseille le sportif de IDSPORT = 12 et IDSPORTIF 16 et IDSPORT = 19 et IDSPORTIF 2  et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    5, 'AAKOUB', 'Linda', 'F',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 12),
            (select ref(s) from Sportif s where s.IDSPORTIF = 16),
			(select ref(s) from Sportif s where s.IDSPORTIF = 19),
			(select ref(s) from Sportif s where s.IDSPORTIF = 2)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);
--//le sportif ABBAS Sophia conseille le sportif de IDSPORT = 10 et IDSPORTIF 6 et IDSPORT = 25  et il est conseillé par le sportif de IDSPORTIF = 3

INSERT INTO Sportif VALUES(
    6, 'ABBAS', 'ABBAS', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 10),
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 25)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);
--//le sportif HADJ Zouhir conseille le sportif de IDSPORT = 30 et IDSPORTIF 9  et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    7, 'HADJ', 'Zouhir', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 30),
            (select ref(s) from Sportif s where s.IDSPORTIF = 9)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif HAMADI Hani conseille le sportif de IDSPORT = 30 et IDSPORTIF = 9 IDSPORTIF = 34 et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    8, 'HAMADI', 'Hani', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 30),
            (select ref(s) from Sportif s where s.IDSPORTIF = 9),
			(select ref(s) from Sportif s where s.IDSPORTIF = 34)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);
--//le sportif ABDELMOUMEN Nadia conseille le sportif de IDSPORT = 67 et IDSPORTIF = 6 IDSPORTIF = 34 et IDSPORTIF = 4  et il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    9, 'ABDELMOUMEN', 'Nadia', 'F',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 67),
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 34),
			(select ref(s) from Sportif s where s.IDSPORTIF = 4)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);
--//le sportif ABAD Abdelhamid conseille le sportif de IDSPORT = 9 et IDSPORTIF = 6 IDSPORTIF = 5 et IDSPORTIF = 13  et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    10, 'ABAD', 'Abdelhamid', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 9),
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 5),
			(select ref(s) from Sportif s where s.IDSPORTIF = 13)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);
--//le sportif ABAYAHIA Amine conseille le sportif de IDSPORT = 97 et IDSPORTIF = 6 IDSPORTIF = 50 et il est conseillé par le sportif de IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    11, 'ABAYAHIA', 'Amine', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 97),
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 50)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);
--//le sportif ABBACI Riad conseille le sportif de IDSPORT = 17 et IDSPORTIF = 8  et il est conseillé par le sportif de IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    12, 'ABBACI', 'Riad', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 17),
            (select ref(s) from Sportif s where s.IDSPORTIF = 8)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);
--//le sportif ABBACI Mohamed conseille le sportif de IDSPORT = 17 et IDSPORTIF = 8  et IDSPORTIF = 88 et IDSPORTIF = 5 il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    13, 'ABBACI', 'Mohamed', 'M',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 17),
            (select ref(s) from Sportif s where s.IDSPORTIF = 7),
			(select ref(s) from Sportif s where s.IDSPORTIF = 88),
			(select ref(s) from Sportif s where s.IDSPORTIF = 5)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif ABDELOUAHAB Lamia conseille le sportif de IDSPORT = 44 et IDSPORTIF = 16  et IDSPORTIF = 8 et  il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    14, 'ABDELOUAHAB', 'Lamia', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 44),
            (select ref(s) from Sportif s where s.IDSPORTIF = 16),
			(select ref(s) from Sportif s where s.IDSPORTIF = 8)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif ABDEMEZIANE Majid conseille le sportif de IDSPORT = 15 et IDSPORTIF = 1  et IDSPORTIF = 8 et IDSPORTIF = 98 et  il est conseillé par le sportif de IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    15, 'ABDEMEZIANE', 'Majid', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 15),
            (select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 8),
			(select ref(s) from Sportif s where s.IDSPORTIF = 98)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);

--//le sportif BENOUADAH Lamine conseille le sportif de IDSPORT = 15 et IDSPORTIF = 1  et IDSPORTIF = 8 et IDSPORTIF = 98 et  il est conseillé par le sportif de IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    16, 'BENOUADAH', 'Lamine', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 15),
            (select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 9),
			(select ref(s) from Sportif s where s.IDSPORTIF = 98)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);

--//le sportif ACHAIBOU Rachid conseille le sportif de IDSPORT = 1 et IDSPORTIF = 18  et IDSPORTIF = 69 et IDSPORTIF = 98 et IDSPORTIF = 75 et  il est conseillé par le sportif de IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    17, 'ACHAIBOU', 'Rachid', 'M',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 1),
            (select ref(s) from Sportif s where s.IDSPORTIF = 18),
			(select ref(s) from Sportif s where s.IDSPORTIF = 69),
			(select ref(s) from Sportif s where s.IDSPORTIF = 98),
			(select ref(s) from Sportif s where s.IDSPORTIF = 75)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif HOSNI Leila conseille le sportif de IDSPORT = 1 et IDSPORTIF = 8  et IDSPORTIF = 6 et IDSPORTIF = 9 et IDSPORTIF = 7 et  il est conseillé par le sportif de IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    18, 'HOSNI', 'Leila', 'F',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 1),
            (select ref(s) from Sportif s where s.IDSPORTIF = 8),
			(select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 9),
			(select ref(s) from Sportif s where s.IDSPORTIF = 7)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif ABERKANE Adel conseille le sportif de IDSPORT = 19 et IDSPORTIF = 68  et IDSPORTIF = 16 et IDSPORTIF = 93 et  il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    19, 'ABERKANE', 'Adel', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 19),
            (select ref(s) from Sportif s where s.IDSPORTIF = 68),
			(select ref(s) from Sportif s where s.IDSPORTIF = 16),
			(select ref(s) from Sportif s where s.IDSPORTIF = 93)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);
--//le sportif AZOUG Racim conseille le sportif de IDSPORT = 19 et IDSPORTIF = 8  et IDSPORTIF = 11  il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    20, 'AZOUG', 'Racim', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 19),
            (select ref(s) from Sportif s where s.IDSPORTIF = 8),
			(select ref(s) from Sportif s where s.IDSPORTIF = 11)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif BABACI Mourad conseille le sportif de IDSPORT = 24 et IDSPORTIF = 18  et IDSPORTIF = 1 et IDSPORTIF = 17  il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    21, 'BABACI', 'Mourad', 'M',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 24),
            (select ref(s) from Sportif s where s.IDSPORTIF = 18),
			(select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 17)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif BAKIR Ayoub conseille le sportif de IDSPORT = 24 et IDSPORTIF = 76  et IDSPORTIF = 15 et IDSPORTIF = 7 et IDSPORTIF = 17  il est conseillé par le sportif de IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    22, 'BAKIR', 'Ayoub', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 24),
            (select ref(s) from Sportif s where s.IDSPORTIF = 76),
			(select ref(s) from Sportif s where s.IDSPORTIF = 15),
			(select ref(s) from Sportif s where s.IDSPORTIF = 7),
			(select ref(s) from Sportif s where s.IDSPORTIF = 17)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);

--//le sportif BEHADI Youcef conseille le sportif de IDSPORT = 4 et IDSPORTIF = 6  et IDSPORTIF = 1 et IDSPORTIF = 71 et IDSPORTIF = 17  il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    23, 'BEHADI', 'Youcef', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 4),
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 71),
			(select ref(s) from Sportif s where s.IDSPORTIF = 17)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif AMARA Nassima conseille le sportif de IDSPORT = 4 et IDSPORTIF = 63  et IDSPORTIF = 87 et IDSPORTIF = 1 et il est conseillé par le sportif de IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    24, 'AMARA', 'Nassima', 'F',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 4),
            (select ref(s) from Sportif s where s.IDSPORTIF = 63),
			(select ref(s) from Sportif s where s.IDSPORTIF = 87),
			(select ref(s) from Sportif s where s.IDSPORTIF = 1)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif AROUEL Lyes conseille le sportif de IDSPORT = 14 et IDSPORTIF = 3  et IDSPORTIF = 8 et IDSPORTIF = 11  et IDSPORTIF = 19 et il est conseillé par le sportif de IDSPORTIF = 9
INSERT INTO Sportif VALUES(
    25, 'AROUEL', 'Lyes', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 14),
            (select ref(s) from Sportif s where s.IDSPORTIF = 3),
			(select ref(s) from Sportif s where s.IDSPORTIF = 8),
			(select ref(s) from Sportif s where s.IDSPORTIF = 11),
			(select ref(s) from Sportif s where s.IDSPORTIF = 19)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 9)
);

--//le sportif BAALI Leila conseille le sportif de IDSPORT = 4 et IDSPORTIF = 31  et IDSPORTIF = 58 et IDSPORTIF = 16  et il est conseillé par le sportif de IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    26, 'BAALI', 'Leila', 'F',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 4),
            (select ref(s) from Sportif s where s.IDSPORTIF = 31),
			(select ref(s) from Sportif s where s.IDSPORTIF = 58),
			(select ref(s) from Sportif s where s.IDSPORTIF = 16)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);

--//le sportif BADI Hatem conseille le sportif de IDSPORT = 4 et IDSPORTIF = 1  et IDSPORTIF = 5 et IDSPORTIF = 16 et IDSPORTIF = 6  et il est conseillé par le sportif de IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    27, 'BADI', 'Hatem', 'F',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 4),
            (select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 5),
			(select ref(s) from Sportif s where s.IDSPORTIF = 16),
			(select ref(s) from Sportif s where s.IDSPORTIF = 6)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif RABAHI Rabah conseille le sportif de IDSPORT = 41 et IDSPORTIF = 13  et IDSPORTIF = 15 et IDSPORTIF = 1  et il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    28, 'RABAHI', 'Rabah', 'F',40,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 41),
            (select ref(s) from Sportif s where s.IDSPORTIF = 13),
			(select ref(s) from Sportif s where s.IDSPORTIF = 15),
			(select ref(s) from Sportif s where s.IDSPORTIF = 1)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif ROUSSELI Lamice conseille le sportif de IDSPORT = 41 et IDSPORTIF = 33  et IDSPORTIF = 1 et IDSPORTIF = 25 et IDSPORTIF = 17  et il est conseillé par le sportif de IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    29, 'ROUSSELI', 'Lamice', 'F',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 41),
            (select ref(s) from Sportif s where s.IDSPORTIF = 33),
			(select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 25),
			(select ref(s) from Sportif s where s.IDSPORTIF = 17)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);


--//le sportif CHIKHI Nidal conseille le sportif de IDSPORT = 1 et IDSPORTIF = 3  et IDSPORTIF = 14 et IDSPORTIF = 2  et il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    30, 'CHIKHI', 'Nidal', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 1),
            (select ref(s) from Sportif s where s.IDSPORTIF = 3),
			(select ref(s) from Sportif s where s.IDSPORTIF = 14),
			(select ref(s) from Sportif s where s.IDSPORTIF = 2)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif SETIHA Moustapha conseille le sportif de IDSPORT = 11 et IDSPORTIF = 31  et IDSPORTIF = 1 et IDSPORTIF = 12 et IDSPORTIF = 22  et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    31, 'SETIHA', 'Moustapha', 'M',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 11),
            (select ref(s) from Sportif s where s.IDSPORTIF = 31),
			(select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 12),
			(select ref(s) from Sportif s where s.IDSPORTIF = 22)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif COTERI Daouad conseille le sportif de IDSPORT = 9 et IDSPORTIF = 11  et IDSPORTIF = 14 et IDSPORTIF = 17 et IDSPORTIF = 22  et il est conseillé par le sportif de IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    32, 'COTERI', 'Daouad', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 9),
            (select ref(s) from Sportif s where s.IDSPORTIF = 11),
			(select ref(s) from Sportif s where s.IDSPORTIF = 14),
			(select ref(s) from Sportif s where s.IDSPORTIF = 17),
			(select ref(s) from Sportif s where s.IDSPORTIF = 22)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);

--//le sportif RAMELI Sami conseille le sportif de IDSPORT = 39 et IDSPORTIF = 14  et IDSPORTIF = 10 et IDSPORTIF = 7  et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    33, 'RAMELI', 'Sami', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 39),
            (select ref(s) from Sportif s where s.IDSPORTIF = 14),
			(select ref(s) from Sportif s where s.IDSPORTIF = 10),
			(select ref(s) from Sportif s where s.IDSPORTIF = 7)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif LEHIRACHE Oussama conseille le sportif de IDSPORT = 76 et IDSPORTIF = 4  et IDSPORTIF = 19 et IDSPORTIF = 17  et il est conseillé par le sportif de IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    34, 'LEHIRACHE', 'Oussama', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 76),
            (select ref(s) from Sportif s where s.IDSPORTIF = 4),
			(select ref(s) from Sportif s where s.IDSPORTIF = 19),
			(select ref(s) from Sportif s where s.IDSPORTIF = 17)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);

--//le sportif TERIKI Yacine conseille le sportif de IDSPORT = 6 et IDSPORTIF = 43  et IDSPORTIF = 1 et IDSPORTIF = 16 et IDSPORTIF = 66  et il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    35, 'TERIKI', 'Yacine', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
            (select ref(s) from Sportif s where s.IDSPORTIF = 43),
			(select ref(s) from Sportif s where s.IDSPORTIF = 1),
			(select ref(s) from Sportif s where s.IDSPORTIF = 16),
			(select ref(s) from Sportif s where s.IDSPORTIF = 66)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif DJELOUDANE Zinedine conseille le sportif de IDSPORT = 66 et IDSPORTIF = 4  et IDSPORTIF = 31 et IDSPORTIF = 6 et IDSPORTIF = 96  et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    36, 'DJELOUDANE', 'Zinedine', 'M',28,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 66),
            (select ref(s) from Sportif s where s.IDSPORTIF = 4),
			(select ref(s) from Sportif s where s.IDSPORTIF = 31),
			(select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 96)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);


--//le sportif LAZARI Zinedine conseille le sportif de IDSPORT = 76 et IDSPORTIF = 45  et IDSPORTIF = 3 et IDSPORTIF = 16 et IDSPORTIF = 6  et il est conseillé par le sportif de IDSPORTIF = 44
INSERT INTO Sportif VALUES(
    37, 'LAZARI', 'Malika', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 76),
            (select ref(s) from Sportif s where s.IDSPORTIF = 45),
			(select ref(s) from Sportif s where s.IDSPORTIF = 3),
			(select ref(s) from Sportif s where s.IDSPORTIF = 16),
			(select ref(s) from Sportif s where s.IDSPORTIF = 6)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 44)
);


--//le sportif MESSOUNI Ismail conseille le sportif de IDSPORT = 6 et IDSPORTIF = 5  et IDSPORTIF = 3 et IDSPORTIF = 6 et IDSPORTIF = 8  et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    38, 'LAZARI', 'Malika', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
            (select ref(s) from Sportif s where s.IDSPORTIF = 5),
			(select ref(s) from Sportif s where s.IDSPORTIF = 3),
			(select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 8)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif MORELI Otheman conseille le sportif de IDSPORT = 16 et IDSPORTIF = 15  et IDSPORTIF = 13 et IDSPORTIF = 17 et IDSPORTIF = 18  et il est conseillé par le sportif de IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    39, 'MORELI', 'Otheman', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 16),
            (select ref(s) from Sportif s where s.IDSPORTIF = 15),
			(select ref(s) from Sportif s where s.IDSPORTIF = 13),
			(select ref(s) from Sportif s where s.IDSPORTIF = 17),
			(select ref(s) from Sportif s where s.IDSPORTIF = 18)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);

--//le sportif FATAHI Majid conseille le sportif de IDSPORT = 26 et IDSPORTIF = 25  et IDSPORTIF = 23 et IDSPORTIF =27 et IDSPORTIF = 28  et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    40, 'FATAHI', 'Majid', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 26),
            (select ref(s) from Sportif s where s.IDSPORTIF = 25),
			(select ref(s) from Sportif s where s.IDSPORTIF = 23),
			(select ref(s) from Sportif s where s.IDSPORTIF = 27),
			(select ref(s) from Sportif s where s.IDSPORTIF = 28)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif DELHOUME Elina Conseille le sportif de IDSPORT = 36 et IDSPORTIF = 45  et IDSPORTIF = 53 et IDSPORTIF =67 et IDSPORTIF = 78  et il est conseillé par le sportif de IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    41, 'DELHOUME', 'Elina', 'F',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 36),
            (select ref(s) from Sportif s where s.IDSPORTIF = 45),
			(select ref(s) from Sportif s where s.IDSPORTIF = 53),
			(select ref(s) from Sportif s where s.IDSPORTIF = 67),
			(select ref(s) from Sportif s where s.IDSPORTIF = 78)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif BEHADI Nadir conseille le sportif de IDSPORT = 36 et IDSPORTIF = 45  et IDSPORTIF = 53 et IDSPORTIF =67 et IDSPORTIF = 78  et il est conseillé par le sportif de IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    42, 'BEHADI', 'Nadir', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 46),
            (select ref(s) from Sportif s where s.IDSPORTIF = 45),
			(select ref(s) from Sportif s where s.IDSPORTIF = 43),
			(select ref(s) from Sportif s where s.IDSPORTIF = 47),
			(select ref(s) from Sportif s where s.IDSPORTIF = 8)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif MATI Dalia conseille le sportif de IDSPORTIF = 5  et IDSPORTIF = 3 et IDSPORTIF =7 et IDSPORTIF = 8  et il est conseillé par le sportif de IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    43, 'MATI', 'Dalia', 'F',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 5),
			(select ref(s) from Sportif s where s.IDSPORTIF = 3),
			(select ref(s) from Sportif s where s.IDSPORTIF = 7),
			(select ref(s) from Sportif s where s.IDSPORTIF = 8)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);

--//le sportif ADIBOU Ibrahim conseille le sportif de IDSPORTIF = 95  et IDSPORTIF = 93 et IDSPORTIF =97 et IDSPORTIF = 98  et il est conseillé par le sportif de IDSPORTIF = 21
INSERT INTO Sportif VALUES(
    44, 'ADIBOU', 'Ibrahim', 'M',28,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 95),
			(select ref(s) from Sportif s where s.IDSPORTIF = 93),
			(select ref(s) from Sportif s where s.IDSPORTIF = 97),
			(select ref(s) from Sportif s where s.IDSPORTIF = 98)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 21)
);


--//le sportif CHALI Karim  conseille aucun sportif  et il est conseillé par aucun sportif
INSERT INTO Sportif VALUES(
    45, 'CHALI', 'Karim', 'M',28,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);

--//le sportif DOUDOU Islam  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    46, 'DOUDOU', 'Islam', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif Grine Célina  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    47, 'Grine', 'Célina', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif HEDDI Zohra  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    48, 'HEDDI', 'Zohra', 'F',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif JADI Sandra  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    49, 'JADI', 'Sandra', 'F',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);


--//le sportif KALI Yasser  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    50, 'KALI', 'Yasser', 'F',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif LAJEL Fouad  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    51, 'LAJEL', 'Fouad', 'M',24,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif DANDOUR Rami  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    52, 'DANDOUR', 'Rami', 'M',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);


--//le sportif DEMMERA Houcine  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    53, 'DEMMERA', 'Houcine', 'M',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif ELKABBADJ Mohammed  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    54, 'ELKABBADJ', 'Mohammed', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif FEROLI Omer  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    55, 'FEROLI', 'Omer', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif GUERRAOUI Zohra  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    56, 'GUERRAOUI', 'Zohra', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);


--//le sportif BOUACHA Aziz  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    57, 'BOUACHA', 'Aziz', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif GUITENI Adam  conseille aucun sportif  et il est conseillé par le  sportif IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    58, 'GUITENI', 'Adam', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);


--//le sportif KACI Samia  conseille aucun sportif  et il est conseillé par aucun  sportif 
INSERT INTO Sportif VALUES(
    59, 'KACI', 'Samia', 'M',23,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);

--//le sportif TIZEGHAT Badis  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 3 
INSERT INTO Sportif VALUES(
    60, 'TIZEGHAT', 'Badis', 'M',32,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);

--//le sportif LAZARRI Jamel  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 7 
INSERT INTO Sportif VALUES(
    61, 'LAZARRI', 'Jamel', 'M',27,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif BAZOUDI Jaouad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 3 
INSERT INTO Sportif VALUES(
    62, 'BAZOUDI', 'Jaouad', 'M',32,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);

--//le sportif AMANI Fadi  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1 
INSERT INTO Sportif VALUES(
    63, 'AMANI', 'Fadi', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif LANORI Faiza  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1 
INSERT INTO Sportif VALUES(
    64, 'LANORI', 'Faiza', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif CHAADI Mourad  conseille aucun sportif  et il est conseillé par aucun sportif
INSERT INTO Sportif VALUES(
    65, 'CHAADI', 'Mourad', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);

--//le sportif DANDANE Mourad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    66, 'DANDANE', 'Mohamed', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif FATTIMI Dalila  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    67, 'FATTIMI', 'Dalila', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif REGHI Jazia  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    68, 'REGHI', 'Jazia', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif MARADI Hadjer  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    69, 'MARADI', 'Hadjer', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif BELMADI Nadji  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 9
INSERT INTO Sportif VALUES(
    70, 'BELMADI', 'Nadji', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 9)
);

--//le sportif DELAROCHI Racim  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    71, 'DELAROCHI', 'Racim', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);

--//le sportif MARTALI Bouzid  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    72, 'MARTALI', 'Bouzid', 'M',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);

--//le sportif DALLIMI Douad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    73, 'DALLIMI', 'Douad', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);

--//le sportif OUBACHA Adel  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    74, 'OUBACHA', 'Adel', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif SAADI Nihal  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    75, 'SAADI', 'Nihal', 'F',39,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif HALGATTI Camelia  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    76, 'HALGATTI', 'Camelia', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);


--//le sportif HIDDOUCI Farid  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    77, 'HALGATTI', 'Farid', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif CHAOUAH Jamel  conseille aucun sportif  et il est conseillé par aucun sportif 
INSERT INTO Sportif VALUES(
    78, 'CHAOUAH', 'Jamel', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);

--//le sportif HANDI Jaouad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    79, 'CHAOUAH', 'Jamel', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif HOCHET Ramezi  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    80, 'HOCHET', 'Ramezi', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif DROULLONI Jaouida  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    81, 'DROULLONI', 'Jaouida', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif DROULLONI Jaouida  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 14
INSERT INTO Sportif VALUES(
    82, 'HOULEMI', 'Lyes', 'M',40,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 14)
);


--//le sportif LOUATI Ahmed  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    83, 'LOUATI', 'Ahmed', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif HAMARI Anes  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    84, 'HAMARI', 'Anes', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);
--//le sportif LOUATI Ahmed  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    85, 'SALLADj', 'Miloud', 'M',28,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif GALLOTI Boualem  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    86, 'GALLOTI', 'Boualem', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif KASBADJI Fateh  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    87, 'KASBADJI', 'Fateh', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif JENOURI Rachid  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    88, 'JENOURI', 'Rachid', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);


--//le sportif RIHABI Jamel  conseille aucun sportif  et il est conseillé par aucun sportif 
INSERT INTO Sportif VALUES(
    89, 'RIHABI', 'Jamel', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);

--//le sportif DERARNI Nadir  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    90, 'DERARNI', 'Nadir', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif BATERAOUI Zinedine  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 98
INSERT INTO Sportif VALUES(
    91, 'BATERAOUI', 'Zinedine', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 98)
);

--//le sportif HADJI Jamel  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    92, 'HADJI', 'Jamel', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif CAUCHARDI Nabil  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    93, 'CAUCHARDI', 'Nabil', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif LEROUDI Moussa  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    94, 'LEROUDI', 'Moussa', 'M',36,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);


--//le sportif ESTANBOULI Mazine  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    95, 'ESTANBOULI', 'Mazine', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif JANID Lamine  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    96, 'JANID', 'Lamine', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif BONHOMMANE Lamine  conseille aucun sportif  et il est conseillé par aucun sportif
INSERT INTO Sportif VALUES(
    97, 'BONHOMMANE', 'Bassim', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);


--//le sportif RIADI Walid  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    98, 'RIADI', 'Walid', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif BONETI Djalal  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    99, 'BONETI', 'Djalal', 'M',32,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    NULL
);


--//le sportif LESOIFI Djamil  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 9
INSERT INTO Sportif VALUES(
    100, 'LESOIFI', 'Djamil', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 9)
);

--//le sportif FATAHI Majid conseille le sportif de IDSPORT = 2 et IDSPORTIF = 6  et IDSPORTIF = 9 et IDSPORTIF = 2  et il est conseillé par le sportif de IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    101, 'SWAMI', 'Esslam', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 2),
            (select ref(s) from Sportif s where s.IDSPORTIF = 6),
			(select ref(s) from Sportif s where s.IDSPORTIF = 9),
			(select ref(s) from Sportif s where s.IDSPORTIF = 2)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif DAOUDI Adel conseille le sportif de IDSPORT = 23 et IDSPORTIF = 63  et IDSPORTIF = 93 et IDSPORTIF = 23  et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    102, 'DAOUDI', 'Adel', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 23),
            (select ref(s) from Sportif s where s.IDSPORTIF = 63),
			(select ref(s) from Sportif s where s.IDSPORTIF = 93),
			(select ref(s) from Sportif s where s.IDSPORTIF = 23)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif DAOUDI Adel conseille le sportif de IDSPORT = 27 et IDSPORTIF = 67  et IDSPORTIF = 9è et IDSPORTIF = 27  et il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    103, 'LAAMOURI', 'Nasssim', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 27),
            (select ref(s) from Sportif s where s.IDSPORTIF = 67),
			(select ref(s) from Sportif s where s.IDSPORTIF = 97),
			(select ref(s) from Sportif s where s.IDSPORTIF = 27)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif SEHIER Dihia conseille le sportif de IDSPORT = 28 et IDSPORTIF = 69  et IDSPORTIF = 90 et IDSPORTIF = 21  et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    104, 'SEHIER', 'Dihia', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 28),
            (select ref(s) from Sportif s where s.IDSPORTIF = 69),
			(select ref(s) from Sportif s where s.IDSPORTIF = 90),
			(select ref(s) from Sportif s where s.IDSPORTIF = 21)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif STITOUAH Fouad conseille le sportif de IDSPORT = 98 et IDSPORTIF = 99  et IDSPORTIF = 90 et IDSPORTIF = 91  et il est conseillé par le sportif de IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    105, 'STITOUAH', 'Fouad', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 98),
            (select ref(s) from Sportif s where s.IDSPORTIF = 99),
			(select ref(s) from Sportif s where s.IDSPORTIF = 90),
			(select ref(s) from Sportif s where s.IDSPORTIF = 91)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);


--//le sportif BAADI Hani conseille le sportif de IDSPORT = 28 et IDSPORTIF = 69  et IDSPORTIF = 90 et IDSPORTIF = 21  et il est conseillé par le sportif de IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    106, 'BAADI', 'Hani', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 12),
            (select ref(s) from Sportif s where s.IDSPORTIF = 13),
			(select ref(s) from Sportif s where s.IDSPORTIF = 14),
			(select ref(s) from Sportif s where s.IDSPORTIF = 15)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif BOURAS Nazim conseille le sportif de IDSPORT = 12 et IDSPORTIF = 13  et IDSPORTIF = 14 et IDSPORTIF = 15  et il est conseillé par le sportif de IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    107, 'BOURAS', 'Nazim', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 12),
            (select ref(s) from Sportif s where s.IDSPORTIF = 13),
			(select ref(s) from Sportif s where s.IDSPORTIF = 14),
			(select ref(s) from Sportif s where s.IDSPORTIF = 15)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);


--//le sportif AIT AMARA Salim conseille le sportif de IDSPORT = 22 et IDSPORTIF = 23  et IDSPORTIF = 24 et IDSPORTIF = 25  et il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    108, 'AIT AMARA', 'Salim', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 22),
            (select ref(s) from Sportif s where s.IDSPORTIF = 23),
			(select ref(s) from Sportif s where s.IDSPORTIF = 24),
			(select ref(s) from Sportif s where s.IDSPORTIF = 25)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);


--//le sportif SAGOU Bassel conseille le sportif de IDSPORT = 28 et IDSPORTIF = 29  et IDSPORTIF = 20 et IDSPORTIF = 22  et il est conseillé par le sportif de IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    109, 'AIT AMARA', 'Salim', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 28),
            (select ref(s) from Sportif s where s.IDSPORTIF = 29),
			(select ref(s) from Sportif s where s.IDSPORTIF = 20),
			(select ref(s) from Sportif s where s.IDSPORTIF = 22)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif ROULLADI Aissa conseille le sportif de IDSPORT = 8 et IDSPORTIF = 9  et IDSPORTIF = 2 et IDSPORTIF = 22  et il est conseillé par le sportif de IDSPORTIF = 4
INSERT INTO Sportif VALUES(
    110, 'ROULLADI', 'Aissa', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
	        (select ref(s) from Sportif s where s.IDSPORTIF = 8),
            (select ref(s) from Sportif s where s.IDSPORTIF = 9),
			(select ref(s) from Sportif s where s.IDSPORTIF = 2),
			(select ref(s) from Sportif s where s.IDSPORTIF = 22)
	),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif BOUTINE Mohamed  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    111, 'BOUTINE', 'Mohamed', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);


--//le sportif LOUATI Islam  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    112, 'LOUATI', 'Islam', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif AID Naim  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    113, 'AID', 'Naim', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif MICHALIKH Asma  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    114, 'MICHALIKH', 'Asma', 'F',22,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif LEMOUSSI Amine  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    115, 'LEMOUSSI', 'Amine', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif BELIFA Samia  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 8
INSERT INTO Sportif VALUES(
    116, 'BELIFA', 'Samia', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 8)
);


--//le sportif FERRIRA Manel  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    117, 'FERRIRA', 'Manel', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif IGHOLI Lyes  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    118, 'IGHOLI', 'Lyes', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif GUEMEZ Jaouad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    119, 'GUEMEZ', 'Jaouad', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);



--//le sportif LECOM Aissa  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    120, 'LECOM', 'Aissa', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);

--//le sportif HOUAT Aziz  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    121, 'HOUAT', 'Aziz', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif BEQUETA Aicha  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    122, 'BEQUETA', 'Aicha', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);

--//le sportif RATENI Walid  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    123, 'RATENI', 'Walid', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);


--//le sportif TOUAT Yasmine  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    124, 'TOUAT', 'Yasmine', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);

--//le sportif JALONI Aimad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    125, 'JALONI', 'Aimad', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif DEBOUBA Aimad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 6
INSERT INTO Sportif VALUES(
    126, 'DEBOUBA', 'yasser', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 6)
);

--//le sportif GASTAB Chouaib  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    127, 'GASTAB', 'Chouaib', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif GIRONI Younes  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    128, 'GASTAB', 'Chouaib', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);


--//le sportif DABONI Rachid  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    129, 'DABONI', 'Rachid', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);


--//le sportif LACHOUBI Kamel  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    130, 'LACHOUBI', 'Kamel', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);


--//le sportif GALLOI, Nadira  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    131, 'GALLOI', 'Nadira', 'F',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif DORONI, Yanis  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    132, 'DORONI', 'Yanis', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif LENOUCHI Youcef  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    133, 'LENOUCHI', 'Youcef', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif LERICHE Hadi  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    134, 'LERICHE', 'Hadi', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif MANSOUR Lamine  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    135, 'MANSOUR', 'Lamine', 'M',30,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 4)
);

--//le sportif LABOULAIS Fadia  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    136, 'LABOULAIS', 'Fadia', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif DOUDOU Faiza  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    137, 'DOUDOU', 'Faiza', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif MAALEM Lamia  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    138, 'MAALEM', 'Lamia', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif BESNARD Salma  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    139, 'BESNARD', 'Salma', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif BELHAMID Hadjer  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    140, 'BELHAMID', 'Hadjer', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif BOUAAZA Asma  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    141, 'BOUAAZA', 'Asma', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);

--//le sportif CORCHI Melissa  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    142, 'CORCHI', 'Melissa', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);


--//le sportif BELAID Jaouida  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 5
INSERT INTO Sportif VALUES(
    143, 'BELAID', 'Jaouida', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 5)
);


--//le sportif GASMI Souad  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    144, 'GASMI', 'Souad', 'F',26,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif LAAMARA Maria  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    145, 'LAAMARA', 'Maria', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif DABOUB Ramezi  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 3
INSERT INTO Sportif VALUES(
    146, 'DABOUB', 'Ramezi', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 3)
);


--//le sportif HASSINI Nadia  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    147, 'HASSINI', 'Nadia', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


--//le sportif KALOUNE Maria  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 1
INSERT INTO Sportif VALUES(
    148, 'KALOUNE', 'Maria', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 1)
);

--//le sportif BELHAOUA Besma  conseille aucun sportif  et il est conseillé par le sportif IDSPORTIF = 7
INSERT INTO Sportif VALUES(
    149, 'BELHAOUA', 'Besma', 'F',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 7)
);

--//le sportif BELAID Fouad conseille le sportif de IDSPORT = 86 et IDSPORTIF = 85  et IDSPORTIF = 83 et IDSPORTIF = 87 et IDSPORTIF = 88  et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    150, 'BELAID', 'Fouad', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 86),
            (select ref(s) from Sportif s where s.IDSPORTIF = 85),
			(select ref(s) from Sportif s where s.IDSPORTIF = 83),
			(select ref(s) from Sportif s where s.IDSPORTIF = 87),
			(select ref(s) from Sportif s where s.IDSPORTIF = 88)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);

--//le sportif HENDI Mouad conseille le sportif de IDSPORT = 46 et IDSPORTIF = 45  et IDSPORTIF = 43 et IDSPORTIF = 47 et il est conseillé par le sportif de IDSPORTIF = 2
INSERT INTO Sportif VALUES(
    151, 'HENDI', 'Mouad', 'M',25,
    T_set_ref_Arbitrer(),
	T_set_ref_Jouer(),
    T_set_ref_Sportif(
            (select ref(s) from Sportif s where s.IDSPORTIF = 46),
            (select ref(s) from Sportif s where s.IDSPORTIF = 45),
			(select ref(s) from Sportif s where s.IDSPORTIF = 43),
			(select ref(s) from Sportif s where s.IDSPORTIF = 47)
    ),
    T_set_ref_Seance(),
    T_set_ref_Entrainer(),
	T_set_ref_Sport(),
    (select ref(s) from Sportif s where s.IDSPORTIF = 2)
);


                 --//---------------------------->table Sport <-----------------------------------
						 
INSERT INTO Sport VALUES(1,'Basket ball',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						  T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);

INSERT INTO Sport VALUES(2,'Volley ball',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						 T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);

INSERT INTO Sport VALUES(3,'Hand ball',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						 T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);

INSERT INTO Sport VALUES(4,'Tennis',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						 T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);

INSERT INTO Sport VALUES(5,'Hockey',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						 T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
						 
);

INSERT INTO Sport VALUES(6,'Badmington',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						  T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
						
);

INSERT INTO Sport VALUES(7,'Ping pong',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						 T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);


INSERT INTO Sport VALUES(8,'Football',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						 T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);


INSERT INTO Sport VALUES(9,'Boxe',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						 T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);

INSERT INTO Sport VALUES(10,'Basket ball',
						 NULL,
						 T_set_ref_Jouer(),
						 T_set_ref_Seance(),
						 T_set_ref_Entrainer(),
						  T_set_ref_Sportif(),
						 T_set_ref_Gymnass()
);


              --//---------------------------->table Entrainer <-----------------------------------
			  
			  
			  
--// lentraineur de IDSPORTIF = 1 entraine le sport de IDSPORT = 1 
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 1),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 1)
											)
							);
--// lentraineur de IDSPORTIF = 1 entraine le sport de IDSPORT = 2 
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 1),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							 
							);
							
--// lentraineur de IDSPORTIF = 1 entraine le sport de IDSPORT = 3 
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 1),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 3)
											)
							 
							);
--// lentraineur de IDSPORTIF = 1 entraine le sport de IDSPORT = 5 
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 1),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 5)
											)
							);
--// lentraineur de IDSPORTIF = 1 entraine le sport de IDSPORT = 6 
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 1),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							 
							);

--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 1 
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 1)
											)
							);
--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 3
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 3)
											)
							);
--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 4
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 4)
											)
							);
--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 5
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 5)
											)
							);
--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 2 entraine le sport de IDSPORT = 9
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 2),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 9)
											)
							);
--// lentraineur de IDSPORTIF = 3 entraine le sport de IDSPORT = 1
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 3),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 1)
											)
							);
--// lentraineur de IDSPORTIF = 3 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 3),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// lentraineur de IDSPORTIF = 3 entraine le sport de IDSPORT = 3
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 3),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 3)
											)
							);
--// lentraineur de IDSPORTIF = 3 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 3),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 4 entraine le sport de IDSPORT = 1
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 4),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 1)
											)
							);
--// lentraineur de IDSPORTIF = 4 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 4),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 4 entraine le sport de IDSPORT = 9
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 4),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 9)
											)
							);
--// lentraineur de IDSPORTIF = 4 entraine le sport de IDSPORT = 5
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 4),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 5)
											)
							);
--// lentraineur de IDSPORTIF = 6 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 6),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 6 entraine le sport de IDSPORT = 9
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 6),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 9)
											)
							);
							
--// lentraineur de IDSPORTIF = 7 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 7),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// lentraineur de IDSPORTIF = 7 entraine le sport de IDSPORT = 3
INSERT INTO Entrainer VALUES(
                              (select ref(b) from Sportif b where b.IDSPORTIF = 7),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 3)
											)
							);
--// lentraineur de IDSPORTIF = 7 entraine le sport de IDSPORT = 5
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 7),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 5)
											)
							);
--// lentraineur de IDSPORTIF = 7 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 7),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 29 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 29),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 30 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 30),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 31 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 31),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 32 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 32),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 35 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 35),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 35 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 35),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 36 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 36),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 38 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 38),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 40 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 40),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 40 entraine le sport de IDSPORT = 7
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 40),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// lentraineur de IDSPORTIF = 48 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 48),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 50 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 50),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 56 entraine le sport de IDSPORT = 6
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 56),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 6)
											)
							);
--// lentraineur de IDSPORTIF = 57 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 57),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// lentraineur de IDSPORTIF = 57 entraine le sport de IDSPORT = 4
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 57),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 4)
											)
							);
--// lentraineur de IDSPORTIF = 58 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 58),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// lentraineur de IDSPORTIF = 58 entraine le sport de IDSPORT = 4
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 58),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 4)
											)
							);
--// lentraineur de IDSPORTIF = 59 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 59),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// l'entraineur de IDSPORTIF = 59 entraine le sport de IDSPORT = 4
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 59),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 4)
											)
							);
--// l'entraineur de IDSPORTIF = 60 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 60),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// l'entraineur de IDSPORTIF = 60 entraine le sport de IDSPORT = 4
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 60),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 4)
											)
							);
--// l'entraineur de IDSPORTIF = 60 entraine le sport de IDSPORT = 4
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 60),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 7)
											)
							);
--// l'entraineur de IDSPORTIF = 61 entraine le sport de IDSPORT = 2
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 61),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 2)
											)
							);
--// l'entraineur de IDSPORTIF = 61 entraine le sport de IDSPORT = 4
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 61),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 4)
											)
							);
--// l'entraineur de IDSPORTIF = 149 entraine le sport de IDSPORT = 1
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 149),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 1)
											)
							);
--// l'entraineur de IDSPORTIF = 151 entraine le sport de IDSPORT = 1
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 151),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 1)
											)
							);
--// l'entraineur de IDSPORTIF = 151 entraine le sport de IDSPORT = 3
INSERT INTO Entrainer VALUES(
                             (select ref(b) from Sportif b where b.IDSPORTIF = 151),
						     T_set_ref_Sport(
							                (select ref(a) from Sport a where a.IDSPORT = 1)
											)
							);



         --//---------------------------->tabl Jouer <-----------------------------------
 
--// le sportif de IDSPORTIF = 1 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 1)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
 
--// le sportif de IDSPORTIF = 1 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 1)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 1 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 1)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);						
--// le sportif de IDSPORTIF = 2 joue le sport de IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 2)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1)
							                )
							);
--// le sportif de IDSPORTIF = 2 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 2)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);						
--// le sportif de IDSPORTIF = 2 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 2)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 2 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 2)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 3 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 3)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 3 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 3)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 4 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 4)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 4 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 4)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 5 joue le sport de IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 5)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1)
							                )
							);
--// le sportif de IDSPORTIF = 5 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 5)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 5 joue le sport de IDSPORT = 6
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 5)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
							                )
							);
--// le sportif de IDSPORTIF = 5 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 5)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 6 joue le sport de IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 6)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1)
							                )
							);
--// le sportif de IDSPORTIF = 6 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 6)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 6 joue le sport de IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 6)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3)
							                )
							);
--// le sportif de IDSPORTIF = 6 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 6)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 7 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 7)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 7 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 7)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 7 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 7)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);					
--// le sportif de IDSPORTIF = 9 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 9)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 9 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 9)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
							
--// le sportif de IDSPORTIF = 9 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 9)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
							                )
							);
--// le sportif de IDSPORTIF = 10 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 10)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 10 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 10)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 10 joue le sport de IDSPORT = 6
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 10)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
							                )
							);
--// le sportif de IDSPORTIF = 10 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 10)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 11 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 11)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 11 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 11)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 11 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 11)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 12 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 12)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 12 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 12)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 12 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 12)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 13 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 13)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 13 joue le sport de IDSPORT = 6
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 13)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
							                )
							);
--// le sportif de IDSPORTIF = 13 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 13)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 14 joue le sport de IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 14)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1)
							                )
							);
--// le sportif de IDSPORTIF = 14 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 14)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 14 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 14)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 15 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 15)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 15 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 15)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 15 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 15)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 16 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 16)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 16 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 16)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 17 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 17)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 17 joue le sport de IDSPORT = 6
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 17)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
							                )
							);
--// le sportif de IDSPORTIF = 17 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 17)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 18 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 18)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 19 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 19)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 19 joue le sport de IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 19)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3)
							                )
							);
--// le sportif de IDSPORTIF = 19 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 19)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 20 joue le sport de IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 20)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1)
							                )
							);
--// le sportif de IDSPORTIF = 20 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 20)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 20 joue le sport de IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 20)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3)
							                )
							);
--// le sportif de IDSPORTIF = 20 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 20)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 20 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 20)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 21 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 21)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 21 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 21)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 21 joue le sport de IDSPORT = 6
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 21)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
							                )
							);
							
--// le sportif de IDSPORTIF = 21 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 21)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 22 joue le sport de IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 22)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1)
							                )
							);
--// le sportif de IDSPORTIF = 22 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 22)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 22 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 22)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 22 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 22)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 23 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 23)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 23 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 23)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
							                )
							);
--// le sportif de IDSPORTIF = 23 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 23)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
							                )
							);
--// le sportif de IDSPORTIF = 24 joue le sport de IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 24)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1)
							                )
							);
--// le sportif de IDSPORTIF = 24 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 24)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 24 joue le sport de IDSPORT = 6
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 24)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
							                )
							);
--// le sportif de IDSPORTIF = 24 joue le sport de IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 24)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 7)
							                )
							);
--// le sportif de IDSPORTIF = 25 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 25)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
							                )
							);
--// le sportif de IDSPORTIF = 25 joue le sport de IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 25)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3)
						                      )
							);
							    
--// le sportif de IDSPORTIF = 25 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 25)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
						                      )
							);
--// le sportif de IDSPORTIF = 25 joue le sport de IDSPORT = 6
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 25)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
						                      )
							);
--// le sportif de IDSPORTIF = 25 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 25)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 26 joue le sport de IDSPORT = 2
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 26)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2)
						                      )
							);
--// le sportif de IDSPORTIF = 26 joue le sport de IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 26)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3)
						                      )
							);
--// le sportif de IDSPORTIF = 26 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 26)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
						                      )
							);
--// le sportif de IDSPORTIF = 26 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 26)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 6)
						                      )
							);
--// le sportif de IDSPORTIF = 27 joue le sport de IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 4 et IDSPORT = 6 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 27)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 28 joue le sport de IDSPORT = 1 et IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 28)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 29 joue le sport de IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 6 et IDSPORT = 7 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 29)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
							
--// le sportif de IDSPORTIF = 30 joue le sport de IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 30)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 31 joue le sport de IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 6 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 31)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 32 joue le sport de IDSPORT = 1 et IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 6 et IDSPORT = 7 IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 32)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 33 joue le sport de IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 6 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 33)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 34 joue le sport de IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 34)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 35 joue le sport de IDSPORT = 1 et IDSPORT = 2 et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 35)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
							
--// le sportif de IDSPORTIF = 36 joue le sport de IDSPORT = 1 et IDSPORT = 2 et  IDSPORT = 7 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 36)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 2),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 37 joue le sport de IDSPORT = 2  
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 37)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 2)
						                      )
							);
--// le sportif de IDSPORTIF = 38 joue le sport de IDSPORT = 1 et IDSPORT = 2 et  IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 38)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 39 joue le sport de IDSPORT = 3 et IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 39)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 40 joue le sport de IDSPORT = 1 et IDSPORT = 3 et IDSPORT = 6 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 40)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 41 joue le sport de IDSPORT = 4 et IDSPORT = 6  
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 41)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 6)
						                      )
							);
--// le sportif de IDSPORTIF = 42 joue le sport de IDSPORT = 4 et IDSPORT = 6 et IDSPORT = 8  
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 42)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 43 joue le sport de IDSPORT = 4 et IDSPORT = 6  et IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 43)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 6)
						                      )
							);
--// le sportif de IDSPORTIF = 44 joue le sport de IDSPORT = 1 et IDSPORT = 7  et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 44)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 45 joue le sport de IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 45)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
							
--// le sportif de IDSPORTIF = 46 joue le sport de IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 46)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 47 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 47)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 4)
						                      )
							);
--// le sportif de IDSPORTIF = 48 joue le sport de IDSPORT = 1 et IDSPORT = 6 et IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 48)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
--// le sportif de IDSPORTIF = 49 joue le sport de IDSPORT = 1 et IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 49)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
--// le sportif de IDSPORTIF = 50 joue le sport de IDSPORT = 1 et IDSPORT = 6 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 50)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 51 joue le sport de IDSPORT = 1 et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 51)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 52 joue le sport de IDSPORT = 1 et IDSPORT = 6 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 52)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 53 joue le sport de IDSPORT = 1 et IDSPORT = 6 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 53)
											),
							   T_set_ref_Sport(
							                    (select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 54 joue le sport de IDSPORT = 6 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 54)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 55 joue le sport de IDSPORT = 6 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 55)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 56 joue le sport de IDSPORT = 1 et IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 56)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
--// le sportif de IDSPORTIF = 57 joue le sport de IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 57)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 58 joue le sport de IDSPORT = 1 et IDSPORT = 6 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 58)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 59 joue le sport de IDSPORT = 1 et IDSPORT = 6 et IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 59)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
--// le sportif de IDSPORTIF = 60 joue le sport de IDSPORT = 3 et IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 60)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 61 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 61)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 62 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 62)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);					
--// le sportif de IDSPORTIF = 63 joue le sport de IDSPORT = 1 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 63)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 64 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 64)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
						                      )
							);
--// le sportif de IDSPORTIF = 65 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 65)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 66 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 66)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 67 joue le sport de IDSPORT = 3 et IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 67)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 4)
						                      )
							);
							
--// le sportif de IDSPORTIF = 68 joue le sport de IDSPORT = 3 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 68)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3)
						                      )
							);
							
--// le sportif de IDSPORTIF = 69 joue le sport de IDSPORT = 3 et IDSPORT = 1 et IDSPORT = 7 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 69)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
							
--// le sportif de IDSPORTIF = 70 joue le sport de IDSPORT = 8 et IDSPORT = 7 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 70)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
--// le sportif de IDSPORTIF = 71 joue le sport de IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 71)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 72 joue le sport de IDSPORT = 3 et IDSPORT = 4 et IDSPORT = 6 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 72)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 73 joue le sport de IDSPORT = 8 et IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 73)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 74 joue le sport de IDSPORT = 8 et IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 74)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 75 joue le sport de IDSPORT = 3 et IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 75)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7)
						                      )
							);
--// le sportif de IDSPORTIF = 76 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 76)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
						                      )
							);
--// le sportif de IDSPORTIF = 76 joue le sport de IDSPORT = 1 et IDSPORT = 7 et IDSPORT  = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 77)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 78 joue le sport de IDSPORT  = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 78)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
--// le sportif de IDSPORTIF = 79 joue le sport de IDSPORT  = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 79)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);
							
--// le sportif de IDSPORTIF = 80 joue le sport de IDSPORT = 1 et IDSPORT = 7 et IDSPORT  = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 80)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 82 joue le sport de IDSPORT  = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 82)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
						                      )
							);			            
--// le sportif de IDSPORTIF = 83 joue le sport de IDSPORT = 3 et IDSPORT = 4 et IDSPORT  = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 83)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 84 joue le sport de IDSPORT = 3 et IDSPORT  = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 84)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 85 joue le sport de IDSPORT = 1 et IDSPORT  = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 85)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 86 joue le sport de IDSPORT = 4 et  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 86)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 87 joue le sport de IDSPORT = 4 et  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 87)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 88 joue le sport de IDSPORT = 1 et  IDSPORT = 8 et IDSPORT = 7
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 88)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 7)
											  )
						);
--// le sportif de IDSPORTIF = 89 joue le sport de IDSPORT = 3 et  IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 89)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 90 joue le sport de IDSPORT = 4 et  IDSPORT = 8 
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 90)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 91 joue le sport de IDSPORT = 1 et  IDSPORT = 7  et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 91)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 92 joue le sport de IDSPORT = 6  et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 92)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
						
--// le sportif de IDSPORTIF = 93 joue le sport de  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 93)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 94 joue le sport de IDSPORT = 1  et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 94)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 95 joue le sport de  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 95)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 96 joue le sport de  IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 96)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 97 joue le sport de  IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 97)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 98 joue le sport de IDSPORT = 1  et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 98)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 99 joue le sport de  IDSPORT = 3 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 99)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 100 joue le sport de  IDSPORT = 3 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 100)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 101 joue le sport de  IDSPORT = 3 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 101)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 102 joue le sport de  IDSPORT = 3 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 102)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 103 joue le sport de  IDSPORT = 3 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 103)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 104 joue le sport de  IDSPORT = 3 et IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 104)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 105 joue le sport de IDSPORT = 1  et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 105)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
						
--// le sportif de IDSPORTIF = 106 joue le sport de  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 106)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 107 joue le sport de  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 107)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 108 joue le sport de IDSPORT = 1   et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 108)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 109 joue le sport de IDSPORT = 1   et IDSPORT = 7 et IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 109)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 109 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 109)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 110 joue le sport de IDSPORT = 3   et IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 110)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 111 joue le sport de IDSPORT = 3  et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 111)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 112 joue le sport de IDSPORT = 3  et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 112)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 113 joue le sport de IDSPORT = 4  et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 113)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 114 joue le sport de IDSPORT = 4  et IDSPORT = 6 et IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 114)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 6),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 115 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 115)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 118 joue le sport de IDSPORT = 1  et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 118)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 119 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 119)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 120 joue le sport de IDSPORT = 8 et IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 120)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 121 joue le sport de IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 121)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 122 joue le sport de IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 122)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 123 joue le sport de IDSPORT = 1 et IDSPORT = 3 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 123)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 3),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 124 joue le sport de IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 124)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 125 joue le sport de IDSPORT = 1 et  IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 125)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 7),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 126 joue le sport   IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 126)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 127 joue le sport   IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 127)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 128 joue le sport   IDSPORT = 4 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 128)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 129 joue le sport   IDSPORT = 1 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 129)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 7)
											  )
						);
--// le sportif de IDSPORTIF = 130 joue le sport de  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 130)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 132 joue le sport   IDSPORT = 1 et IDSPORT = 7 et IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 132)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 7)
											  )
						);
--// le sportif de IDSPORTIF = 133 joue le sport de  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 133)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 134 joue le sport de  IDSPORT = 8
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 134)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8)
											  )
						);
--// le sportif de IDSPORTIF = 135 joue le sport de  IDSPORT = 8 et IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 135)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 8),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 136 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 136)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 137 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 137)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 138 joue le sport de  IDSPORT = 4 et IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 138)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
--// le sportif de IDSPORTIF = 139 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 139)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 140 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 140)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 141 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 141)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 142 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 142)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 143 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 143)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 144 joue le sport de  IDSPORT = 4
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 144)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 4)
											  )
						);
--// le sportif de IDSPORTIF = 149 joue le sport de  IDSPORT = 1
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 149)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1)
											  )
						);
--// le sportif de IDSPORTIF = 151 joue le sport de  IDSPORT = 1 et IDSPORT = 3
INSERT INTO Jouer VALUES(
						       T_set_ref_Sportif(
							                    (select ref(a) from Sportif a where a.IDSPORTIF = 151)
											),
							   T_set_ref_Sport(
												(select ref(b) from Sport b where b.IDSPORT = 1),
												(select ref(b) from Sport b where b.IDSPORT = 3)
											  )
						);
						
            --//---------------------------->table Gymnass <-----------------------------------
			
			
--//la salle IDGYMNASE = 1 se trouve a la ville Alger centre
INSERT INTO Gymnass VALUES(1,'Five Gym Club','Boulevard Mohamed 5',200,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Alger centre'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 2 se trouve a la ville Les sources
INSERT INTO Gymnass VALUES(2,'Mina Sport','28 impasse musette les sources',450,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Les sources'),
							T_set_ref_Sport()
							);				
--//la salle IDGYMNASE = 3 se trouve a la ville Belouizdad
INSERT INTO Gymnass VALUES(3,'Aït Saada','Belouizdad',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Belouizdad'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 4 se trouve a la ville Sidi Mhamed
INSERT INTO Gymnass VALUES(4,'Bahri Gym','Rue Mohamed Benzineb',500,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Sidi Mhamed'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 5 se trouve a la ville El Biar
INSERT INTO Gymnass VALUES(5,'Ladies First','3 Rue Diar Naama N° 03',620,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'El Biar'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 6 se trouve a la ville El Mouradia
INSERT INTO Gymnass VALUES(6,'C.T.F Club','Rue Sylvain FOURASTIER',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'El Mouradia'),
							T_set_ref_Sport()
							);		
--//la salle IDGYMNASE = 7 se trouve a la ville Alger centre
INSERT INTO Gymnass VALUES(7,'Body Fitness Center','Rue Rabah Takdjourt',360,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Alger centre'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 8 se trouve a la ville Hydra
INSERT INTO Gymnass VALUES(8,'Club Hydra Forme','Rue de l''Oasis',420,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Hydra'),
							T_set_ref_Sport()
							);								
--//la salle IDGYMNASE = 9 se trouve a la ville Dely Brahim
INSERT INTO Gymnass VALUES(9,'Profitness Dely Brahim','26 Bois des Cars 3',620,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Dely Brahim'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 10 se trouve a la ville Kouba
INSERT INTO Gymnass VALUES(10,'CLUB SIFAKS','Rue Ben Omar 31',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Kouba'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 11 se trouve a la ville El Mouradia
INSERT INTO Gymnass VALUES(11,'Gym ZAAF Club','19 Ave Merabet Athmane',300,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'El Mouradia'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 12 se trouve a la ville Bir Mourad Raïs
INSERT INTO Gymnass VALUES(12,'GYM power','villa N°2, Chemin Said Hamdine',480,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Bir Mourad Raïs'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 13 se trouve a la ville Hydra
INSERT INTO Gymnass VALUES(13,'Icosium sport','Rue ICOSUM',200,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Hydra'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 14 se trouve a la ville Birkhadem
INSERT INTO Gymnass VALUES(14,'GIGA Fitness','res, Rue Hamoum Tahar',500,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Birkhadem'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 15 se trouve a la ville Birkhadem
INSERT INTO Gymnass VALUES(15,'AC Fitness Et Aqua','Lotissement FAHS lot A n 12 parcelle 26',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Birkhadem'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 16 se trouve a la ville El Achour
INSERT INTO Gymnass VALUES(16,'MELIA GYM','Résidence les deux bassins Sahraoui local N° 03',600,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'El Achour'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 17 se trouve a la ville Kouba
INSERT INTO Gymnass VALUES(17,'Sam Gym Power','Rue Mahdoud BENKHOUDJA',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Kouba'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 18 se trouve a la ville Bordj el kiffan
INSERT INTO Gymnass VALUES(18,'AQUAFORTLAND SPA','Bordj el kiffan',450,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Bordj el kiffan'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 19 se trouve a la ville Baba hassenn
INSERT INTO Gymnass VALUES(19,'GoFitness','Lotissement el louz n°264',450,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Baba hassen'),
							T_set_ref_Sport()
							);	
	
--//la salle IDGYMNASE = 20 se trouve a la ville Chéraga
INSERT INTO Gymnass VALUES(20,'Best Body Gym','Cité Alioua Fodil',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Chéraga'),
							T_set_ref_Sport()
							);	
--//la salle IDGYMNASE = 21 se trouve a la ville Alger
INSERT INTO Gymnass VALUES(21,'Power house gym','Cooperative Amina 02 Lot 15',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Alger'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 22 se trouve a la ville Alger
INSERT INTO Gymnass VALUES(22,'POWER ZONE GYM','Chemin Fernane Hanafi',500,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Hussein Dey'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 23 se trouve a la ville Béni Messous
INSERT INTO Gymnass VALUES(23,'World Gym','14 Boulevard Ibrahim Hadjress',520,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Béni Messous'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 24 se trouve a la ville Bordj El Bahr
INSERT INTO Gymnass VALUES(24,'Moving Club','Bordj El Bahri',450,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Bordj El Bahr'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 25 se trouve a la ville Chéraga
INSERT INTO Gymnass VALUES(25,'Tiger gym','Route de Bouchaoui',620,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Chéraga'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 26 se trouve a la ville Mohammadia
INSERT INTO Gymnass VALUES(26,'Lion CrossFit','Centre commercial-Mohamadia mall',600,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Mohammadia'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 27 se trouve a la ville Saoula
INSERT INTO Gymnass VALUES(27,'Étoile sportive','Saoula',350,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'Saoula'),
							T_set_ref_Sport()
							);
--//la salle IDGYMNASE = 28 se trouve a la ville El Harrach
INSERT INTO Gymnass VALUES(28,'Fitness life gym','El Harrach',400,
							T_set_ref_Seance(),
							(select ref(v) from ville v where v.VILLE = 'El Harrach'),
							T_set_ref_Sport()
							);
				--//---------------------------->table Ville <-----------------------------------
				
				
--// la ville Alger Centre Contient la salle IDGYMNASE = 1 et IDGYMNASE = 7 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Alger centre',
                          T_set_ref_Gymnass(
						                     (select ref(g) from gymnass g where g.IDGYMNASE= 1),
										     (select ref(g) from gymnass g where g.IDGYMNASE= 7)
                                           )										   
                         );
--// la ville Les sources Contient la salle  IDGYMNASE = 2 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Les sources',
                          T_set_ref_Gymnass(
										     (select ref(g) from gymnass g where g.IDGYMNASE= 2)
                                           )										   
                         );
--// la ville Belouizdad Contient la salle  IDGYMNASE = 3 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Belouizdad',
                          T_set_ref_Gymnass(
										     (select ref(g) from gymnass g where g.IDGYMNASE= 3)
                                           )										   
                         );
--// la ville Sidi Mhamed Contient la salle  IDGYMNASE = 4 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Sidi Mhamed',
                          T_set_ref_Gymnass(
										     (select ref(g) from gymnass g where g.IDGYMNASE= 4)
                                           )										   
                         );
--// la ville El Biar Contient la salle  IDGYMNASE = 5 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('El Biar',
                          T_set_ref_Gymnass(
										     (select ref(g) from gymnass g where g.IDGYMNASE= 5)
                                           )										   
                         );
--// la ville El Mouradia Contient la salle  IDGYMNASE = 6 et IDGYMNASE = 11 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('El Mouradia',
                          T_set_ref_Gymnass(
										     (select ref(g) from gymnass g where g.IDGYMNASE= 6),
											 (select ref(g) from gymnass g where g.IDGYMNASE= 11)
                                           )										   
                         );
--// la ville Hydra Contient la salle  IDGYMNASE = 8 et IDGYMNASE = 13 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Hydra',
                          T_set_ref_Gymnass(
										     (select ref(g) from gymnass g where g.IDGYMNASE= 8),
											 (select ref(g) from gymnass g where g.IDGYMNASE= 13)
                                           )										   
                         );
--// la ville Dely Brahim Contient la salle  IDGYMNASE = 8 et IDGYMNASE = 13 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Dely Brahim',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 9)
                                           )										   
                         );
--// la ville Kouba Contient la salle  IDGYMNASE = 10 et IDGYMNASE = 17 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Kouba',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 10),
											 (select ref(g) from gymnass g where g.IDGYMNASE= 17)
                                           )										   
                         );
--// la ville Bir Mourad Raïs Contient la salle  IDGYMNASE = 12 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Bir Mourad Raïs',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 12)
                                           )										   
                         );
--// la ville Birkhadem  Contient la salle  IDGYMNASE = 14 et IDGYMNASE = 15 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Birkhadem',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 14),
											 (select ref(g) from gymnass g where g.IDGYMNASE= 15)
                                           )										   
                         );
--// la ville El Achour Contient la salle  IDGYMNASE = 16 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('El Achour',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 16)
                                           )										   
                         );
--// la ville Bordj el kiffan Contient la salle  IDGYMNASE = 18 (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Bordj el kiffan',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 18)
                                           )										   
                         );
--// la ville Baba hassen Contient la salle  IDGYMNASE = 19  (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Baba hassen',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 19)
                                           )										   
                         );
--// la ville Chéraga Contient la salle  IDGYMNASE = 25 et IDGYMNASE = 20  (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Chéraga',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 20),
											 (select ref(g) from gymnass g where g.IDGYMNASE= 25)
                                           )										   
                         );
--// la ville Alger Contient la salle  IDGYMNASE = 21  (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Alger',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 21)
                                           )										   
                         );
--// la ville Hussein Dey Contient la salle  IDGYMNASE = 22  (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Hussein Dey',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 22)
                                           )										   
                         );
--// la ville Béni Messous Contient la salle  IDGYMNASE = 22  (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Béni Messous',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 23)
                                           )										   
                         );
--// la ville Bordj El Bahri Contient la salle  IDGYMNASE = 24  (ils sont mentionées deja dans la table gymnass donc on va l'extraire)
INSERT INTO Ville VALUES('Bordj El Bahri',
                          T_set_ref_Gymnass(
											 (select ref(g) from gymnass g where g.IDGYMNASE= 24)
                                           )										   
                         );
						 
						 
					--//---------------------------->table Seance <-----------------------------------

--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 149 avec le sport IDSPORT = 1
INSERT INTO seance VALUES('SAMEDI', 9.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 149),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 1)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 1 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 9.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 1),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 1 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 11.3, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 1),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 1 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 14.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 1),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 1 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 17.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 1),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 1 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 19.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 1),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('DIMANCHE', 17.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 2),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1)
							(select ref(f) from sport f where f.IDSPORT = 3),
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('DIMANCHE', 19.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 2),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('MARDI', 17.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 2),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('MERCREDI', 17.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 2),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('SAMEDI', 15.3, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 2),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('SAMEDI', 16.3, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 2),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('SAMEDI', 17.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 2),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('JEUDI', 20.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 14.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 18.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 19.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('LUNDI', 20.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 1 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MERCREDI', 17.0, 190,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 1),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 2 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 2),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 3 par l'entraineur IDSPORTIF = 149 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('MERCREDI', 11.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 149),
							(select ref(g) from gymnass g where g.IDGYMNASE = 3),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 3 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 16.3, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 3),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 3 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('JEUDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 3),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 4 par l'entraineur IDSPORTIF = 149 avec le sport IDSPORT = 1
INSERT INTO seance VALUES('VENDREDI', 10.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 149),
							(select ref(g) from gymnass g where g.IDGYMNASE = 4),
							(select ref(f) from sport f where f.IDSPORT = 1)
						 );
--// la seance est faite dans le salle IDGYMNASE = 4 par l'entraineur IDSPORTIF = 5 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MERCREDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 5),
							(select ref(g) from gymnass g where g.IDGYMNASE = 4),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 5 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 16.3, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 5),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 5 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('JEUDI', 19.3, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 5),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 6 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('VENDREDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 6),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 6 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('JEUDI', 17.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 6),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 8 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 8),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 8 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 16.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 8),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 8 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('VENDREDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 8),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 8 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('SAMEDI', 17.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 8),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 8 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('VENDREDI', 14.0, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 8),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 9 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('SAMEDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 9),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 10 par l'entraineur IDSPORTIF = 2 avec le sport IDSPORT = 60
INSERT INTO seance VALUES('SAMEDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 10),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 10 par l'entraineur IDSPORTIF =6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('DIMANCHE', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 10),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 10 par l'entraineur IDSPORTIF =7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 10),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 12 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 12),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 13 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 13),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 13 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MERCREDI', 20.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 13),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 13 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('LUNDI', 17.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 13),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 14 par l'entraineur IDSPORTIF = 149 avec le sport IDSPORT = 1
INSERT INTO seance VALUES('MARDI', 10.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 149),
							(select ref(g) from gymnass g where g.IDGYMNASE = 14),
							(select ref(f) from sport f where f.IDSPORT = 1)
						 );
--// la seance est faite dans le salle IDGYMNASE = 14 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 14),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 15 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 16.3, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 15),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 16 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 16.3, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 16),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 16 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 16),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 16 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 18.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 16),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 16 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 16),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 16 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 12.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 16),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 16 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MERCREDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 16),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 17 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('SAMEDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 17),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 17 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('VENDREDI', 17.3, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 17),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 17 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 17),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 17 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('DIMANCHE', 18.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 17),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 17 par l'entraineur IDSPORTIF = 3 avec le sport IDSPORT = 3
INSERT INTO seance VALUES('MARDI', 20.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 3),
							(select ref(g) from gymnass g where g.IDGYMNASE = 17),
							(select ref(f) from sport f where f.IDSPORT = 3)
						 );
--// la seance est faite dans le salle IDGYMNASE = 17 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MARDI', 17.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 17),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 18 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 16.3, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 18),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 18 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('MARDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 18),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 18 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('MERCREDI', 14.0, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 18),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 18 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('MERCREDI', 16.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 18),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 19 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 16.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 19),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 20 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('MERCREDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 20),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 21 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('LUNDI', 16.3, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 21),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 21 par l'entraineur IDSPORTIF = 60 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('MARDI', 19.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 60),
							(select ref(g) from gymnass g where g.IDGYMNASE = 21),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 21 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MERCREDI', 17.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 21),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 22 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MARDI', 10.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 22),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 24 par l'entraineur IDSPORTIF = 149 avec le sport IDSPORT = 1
INSERT INTO seance VALUES('JEUDI', 9.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 149),
							(select ref(g) from gymnass g where g.IDGYMNASE = 24),
							(select ref(f) from sport f where f.IDSPORT = 1)
						 );
--// la seance est faite dans le salle IDGYMNASE = 24 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('JEUDI', 10.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 24),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 25 par l'entraineur IDSPORTIF = 149 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('DIMANCHE', 18.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 149),
							(select ref(g) from gymnass g where g.IDGYMNASE = 25),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 27 par l'entraineur IDSPORTIF = 57 avec le sport IDSPORT = 2
INSERT INTO seance VALUES('JEUDI', 10.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 57),
							(select ref(g) from gymnass g where g.IDGYMNASE = 27),
							(select ref(f) from sport f where f.IDSPORT = 2)
						 );
--// la seance est faite dans le salle IDGYMNASE = 27 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MERCREDI', 14.0, 120,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 27),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 27 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MERCREDI', 17.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 27),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 149 avec le sport IDSPORT = 1
INSERT INTO seance VALUES('LUNDI', 9.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 149),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 1)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('DIMANCHE', 9.0, 30,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('DIMANCHE', 15.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('DIMANCHE', 16.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 6 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('DIMANCHE', 17.0, 60,
                            (select ref(s) from sportif s where s.IDSPORTIF = 6),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('MARDI', 18.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('SAMEDI', 18.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
--// la seance est faite dans le salle IDGYMNASE = 28 par l'entraineur IDSPORTIF = 7 avec le sport IDSPORT = 5
INSERT INTO seance VALUES('VENDREDI', 18.0, 90,
                            (select ref(s) from sportif s where s.IDSPORTIF = 7),
							(select ref(g) from gymnass g where g.IDGYMNASE = 28),
							(select ref(f) from sport f where f.IDSPORT = 5)
						 );
/*Partie V : Langage d’interrogation de données*/
		
--// requete 1 : Quels sont les sportifs (identifiant, nom et prénom) qui ont un âge entre 20 et 30 ans ?

select s.IDSPORTIF, s.nom, s.prenom,s.age
from sportif s 
where s.age <= 30 and s.age >= 20
order by s.age;

--//requete 2 Afficher la superficie moyenne des gymnases, pour chaque ville.

SELECT distinct v.ville, v.CALCUL_SUP_MOY_GYMNASS() FROM ville v;
--// requete 3: Quels sont les sportifs qui sont des conseillers ?

select s.nom,s.prenom 
from sportif s
where s.conseille is not null
order by s.IDSPORTIF;

--// requete 4 Quels entraîneurs n’entraînent que du hand ball ou du basket ball ?

select distinct deref(e.entrainer_sportif).nom , deref(e.entrainer_sportif).prenom
from entrainer e , table(e.entrainer_sport) t1
where deref(t1.column_value).LIBELLE = 'Hand ball' or
      deref(t1.column_value).LIBELLE = 'Basket ball';
	
--// requete5  Quels sont les sportifs les plus jeunes?

--//----------> sous requete1
SELECT age, COUNT(age) AS age_count 
                       FROM sportif			   
                       GROUP BY age;
//----------> sous requete2
SELECT age, COUNT(age) AS age_count 
                       FROM sportif
                       where age <=30 and age >= 20					   
                       GROUP BY age;
--//----------> sous requete3 

SELECT age, COUNT(age) AS age_count 
FROM sportif 
GROUP BY age
HAVING COUNT(age) = (
    SELECT MAX(age_count)as subquery
    FROM (
        SELECT COUNT(age) AS age_count
        FROM sportif
		where age <=30 and age >= 20
        GROUP BY age
    )
);

--//----------> sous requete4

SELECT nom, prenom 
FROM sportif 
WHERE age IN (
    SELECT age 
    FROM sportif 
    GROUP BY age 
    HAVING COUNT(*) = (
        SELECT MAX(age_count) as subquery
        FROM (
            SELECT COUNT(*) AS age_count		
            FROM sportif 
			where age <=30 and age >= 20	
            GROUP BY age
        )
    )
);


