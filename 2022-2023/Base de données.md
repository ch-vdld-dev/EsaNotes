
Voici une première série d'exercices en vue de vous préparer pour l'examen. La 2è série arrivera au courant de la semaine. Si vous faites consciencieusement ces exercices, il est certain que vous réussirez l'examen.

Créer une base de donnée plaisance et injecter le fichier sql ci-joint.

Voici ce que vous devez faire

##### Ajouter les clés primaires et les clefs d'unicité  
	ALTER TABLE ville ADD PRIMARY KEY (ville_id);
	

##### Ajouter les contraintes de clés étrangères  
	ALTER TABLE ville ADD FOREIGN KEY (ville_pays) REFERENCES pays (pays_id);
	
##### Aucune colonne ne peut-être nulle  
	ALTER TABLE ville ALTER COLUMN ville_nom SET NOT NULL;
	
##### Le code postal d'une ville >= 1000  
	ALTER TABLE ville ADD CONSTRAINT cp CHECK (ville_codepostal >= 1000);

##### Toutes les colonnes modification ont la date et heure courante comme valeur par défaut
	 ALTER TABLE ville ALTER COLUMN modification SET DEFAULT now();

##### Ajouter un trigger 'before_update' pour que lors d'un UPDATE, les colonnes modification prennent la date et heure courante comme valeur  
	CREATE OR REPLACE FUNCTION set_now() RETURNS TRIGGER AS $$
	BEGIN
		NEW.modification = now();
		RETURN NEW;
	END;
	$$ LANGUAGE plpgsql;

	CREATE TRIGGER before_update
		BEFORE UPDATE
		ON adresse
		FOR EACH ROW
		EXECUTE FUNCTION set_now();


###### To do it at once in all tables
	DO $$
	DECLARE
		t text;
	BEGIN
		FOR t IN SELECT table_name FROM information_schema.tables WHERE columns_name = 'modification';
		LOOP
			RAISE NOTICE 'Create trigger for: %', t;
			EXECUTE 'CREATE TRIGGER before_update BEFORE UPDATE ON ' || t || 'FOR EACH ROW
			EXECUTE PROCEDURE set_now();';
		END LOOP;
	END;
	$$ LANGUAGE pspgsql;
	
##### Créer une vue 'vue_client_complet' qui affiche l'adresse complète pour chaque client  
	CREATE OR REPLACE VIEW vue_client_complet AS 
		SELECT c.client_nom, a.adresse_rue, v.ville_nom 
		FROM client c 
		JOIN adresse a ON c.client_addresse = addresse_id
		JOIN ville v on addresse_ville = ville_id
		JOIN pays p on ville_pays = pays_id;

##### Créer une vue 'vue_ville_pays' qui affiche les villes par pays
	CREATE OR REPLACE VIEW view_ville_pays AS SELECT p.pays_nom, v.ville_nom FROM ville v JOIN pays p ON ville_pays = pays_id;

##### Créer une fonction 'estceque_ville_existe (nom_ville, cp_ville, nom_pays)' qui retourne un boolean pour indiquer si la ville existe dans la base de données  
	CREATE OR REPLACE FUNCTION ville_exist(nom_ville VARCHAR, cp INT, nom_pays VARCHAR) AS $$
	BEGIN
		Perfom ville_nom FROM ville JOIN pays ON ville_pays = pays_id 
		WHERE ville_nom = nom_ville
		AND ville_codepostal = cp
		AND pays_nom = nom_pays;
		IF FOUND THEN
			RAISE NOTICE 'Ville trouve';
			RETURN true;
		END IF;
		RETURN false;
	END;
	$$ LANGUAGE plpgsql;
	
select ville_existe('Dinant', 5500, 'Belgique'); 
select ville_existe('Dinant', 5600, 'Belgique');

##### Créer une procédure 'ajout_ville (nom_ville, cp_ville, nom_pays)' qui permet d'ajouter une ville dans un pays si elle n'existe pas  
	CREATE OR REPLACE PROCEDURE ajout_ville (ville VARCHAR, cp INT , pays VARCHAR) AS $$
	DECLARE
		idpays INT;
	BEGIN
		IF ville_exist(ville, cp, pays) THEN
			RAISE INFO ('Ville existe deja')
		ELSE
			SELECT pays_nom INTO idpays FROM pays WHERE pays_nom = nom_pays;
			IF NOT FOUND THEN
				INSERT INTO pays(pays_nom) VALUES (nom_pays);
				SELECT pays_id INTO ipays FROM pays WHERE pays_nom = nom_pays;
			END IF;
			INSERT INTO ville (ville_nom, ville_codepostal, ville_pays) VALUES (nom_ville, cp, idpays);
			RAISE INFO ('ville ajoutée')
		END IF;
	END;
	$$ LANGUAGE plpgsql;	

call ajout_ville ('Charleroi', 6000, 'Belgique');

##### Créer une procédure 'ajout_adresse (nom_rue, nom_ville, cp_ville, nom_pays)' qui permet d'ajouter une adresse si elle n'existe pas. ATTENTION, une même adresse peut se retrouver dans plusieurs villes  
	CREATE OR REPLACE PROCEDURE ajout_adresse(nom_rue VARCHAR, ville VARCHAR, cp INT, pays VARCHAR) AS $$
	DECLARE
		idville INT;
	BEGIN
		SELECT adresse_ville INTO idville FROM adresse 
		JOIN ville ON adresse_ville = ville_id 
		JOIN pays ON ville_pays = pays_id 
		WHERE adresse_rue = nom_rue 
		AND ville_nom = ville 
		AND ville_codepostal = cp AND;
		AND pays_nom = pays;
		IF NOT FOUND THEN
			call ajout_ville;
			SELECT ville_id INTO idville FROM ville WHERE ville_nom = ville;
			INSERT INTO adresse(adresse_rue, adresse_ville) VALUES (ville, idville);
			RAISE INFO 'adresse ajoutee';
		ELSE
			RAISE INFO 'adresse existe deja dans cette ville';
		END IF;
	END;
	$$ LANGUAGE plpgsql


##### Créer une procédure 'ajout_client (nom, prenom)' qui permet d'ajouter un client s'il n'existe pas


##### Créer un trigger 'before_update' et 'before_insert' sur le client qui corrigent le nom et le prénom en mettant la 1è lettre en majuscule et le reste en minuscule.

	CREATE OR REPLACE FUNCTION capitalize() RETURNS TRIGGER
	LANGUAGE PLPGSQL
	AS $$
	BEGIN
		NEW.client_nom = INITCAP(NEW.client_nom);
		NEW.client_prenom = INITCAP(NEW.client_prenom);
		RETURN NEW;
	END;
	$$;

	CREATE OR REPLACE TRIGGER cap_before_update
		BEFORE UPDATE OR INSERT ON client
		FOR EACH ROW
		EXECUTE FUNCTION capitalize();

Examen 2023

Voici une première série d'exercices en vue de vous préparer pour l'examen. La 2è série arrivera au courant de la semaine. Si vous faites consciencieusement ces exercices, il est certain que vous réussirez l'examen.

Créer une base de donnée plaisance et injecter le fichier sql ci-joint.

Voici ce que vous devez faire

Ajouter les clés primaires et les clefs d'unicité  
Ajouter les contraintes de clés étrangères  
Aucune colonne ne peut-être nulle (sauf une qui doit être nulle par défaut : à vous de la trouver)  
Le code postal d'une ville >= 1000  
Toutes les colonnes modification ont la date et heure courante comme valeur par défaut

Créer un trigger 'before_update' pour que lors d'un UPDATE, les colonnes modification prennent la date et heure courante comme valeur  
Créer un trigger 'before_update' et 'before_insert' sur le client qui corrigent le nom et le prénom en mettant la 1è lettre en majuscule et le reste en minuscule.  
  
Créer une vue 'vue_client_complet' qui affiche l'adresse complète pour chaque client  
Créer une vue 'vue_ville_pays' qui affiche les villes par pays  
Créer une vue 'location par client' qui affiche toutes les locations pour tous les clients triées par client + date de location début

Créer une fonction 'estceque_ville_existe (nom_ville, cp_ville, nom_pays)' qui retourne un boolean pour indiquer si la ville existe dans la base de données  
  
Créer une procédure 'ajout_ville (nom_ville, cp_ville, nom_pays)' qui permet d'ajouter une ville dans un pays si elle n'existe pas  
Créer une procédure 'ajout_adresse (nom_rue, nom_ville, cp_ville, nom_pays)' qui permet d'ajouter une adresse si elle n'existe pas. ATTENTION, une même adresse peut se retrouver dans plusieurs villes  
Créer une procédure 'ajout_client (nom, prenom, nom_rue, nom_ville, cp_ville, nom_pays)' qui permet d'ajouter un client s'il n'existe pas  
Créer une procédure 'ajout_location (nom, prenom, nom_bateau, prix, date_location)' qui créée une nouvelle location pour le bateau correspondant. Attention, il faut vérifier si le bateau n'est pas déjà en location et que le bateau peut être loué (bateau_valide)  
Créer une procédure 'fin_location(id_location)' qui est utilisée lorsque le bateau est rendu. Il faut vérifier que id_location correspond bien à une location non clôturée.

Bon travail  
Geoffroy Malherbe

0493 14 21 52
