# Cassandra

## Killrvideo

### Exercise 1

- Creare e usare il keyspace `killrvideo`


      CREATE KEYSPACE killrvideo WITH REPLICATION = { 'class': 'SimpleStrategy', 'replication_factor' : 1 };

      USE killrvideo;

- Creare una tabella `videos` per  memorizzare i video  con le colonne indicate

      CREATE TABLE videos( 
            video_id TIMEUUID, 
            added_date TIMESTAMP,
            description TEXT, 
            title TEXT, 
            user_id uuid,
            PRIMARY KEY (video_id) 
      );

- Caricare dati nella tabella. Fare riferimento al file `/root/labwork/exercise-2/videos.csv`

      COPY videos
      FROM '/root/labwork/exercise-2/videos.csv' 
      WITH HEADER=true;

- Visualizzare i dati con una SELECT

      SELECT * FROM videos; 

- Contare i video

      SELECT COUNT(*) FROM videos; 

- Selezionare 10 video

      SELECT * FROM videos LIMIT 10;

### Exercise 2

- Creare una nuova tabella `videos_by_title_year` per memorizzare video con una  chiave composta. E' buona pratica dare alle tabelle un nome  che rispecchi il modo di interrogarle

      CREATE TABLE videos_by_title_year(  
            title TEXT, 
            added_year int,
            added_date TIMESTAMP,
            description TEXT,
            user_id uuid,
            video_id TIMEUUID,
            PRIMARY KEY ((title, added_year)) 
      );

- Caricare dati nella tabella. Fare riferimento al file `/root/labwork/exercise-3/videos_by_title_year.csv`

      COPY videos_by_title_year
      FROM '/root/labwork/exercise-3/videos_by_title_year.csv' 
      WITH HEADER=true;

- Cercare un video per nome e anno

      SELECT * 
      FROM videos_by_title_year 
      WHERE title = 'Sleepy Grumpy Cat'
      AND added_year = 2015; 


- Cercare video solo per nome o solo per anno (non funziona, vedremo perchè nelle prossime slide)

      SELECT * 
      FROM videos_by_title_year 
      WHERE title = 'Sleepy Grumpy Cat'; 

      SELECT * 
      FROM videos_by_title_year 
      WHERE added_year = 2015; 


### Exercise 3

- Creare una nuova tabella `videos_by_tag_year` per  memorizzare video con una  chiave composta diversa
  - Obiettivo: interrogare la tabella sulla base dei campi tag e year e fare range queries su year
  - ATTENZIONE: un video può essere associate a tanti tag

        CREATE TABLE videos_by_tag_year(  
            tag TEXT, 
            added_year int,
            added_date TIMESTAMP,
            title TEXT, 
            description TEXT,
            user_id uuid,
            video_id TIMEUUID,
            PRIMARY KEY ((tag), added_year, video_id) 
        ) WITH CLUSTERING ORDER BY (added_year DESC);

- Caricare dati nella tabella. Fare riferimento al file `/root/labwork/exercise-4/videos_by_tag_year.csv`

      COPY videos_by_tag_year
      FROM '/root/labwork/exercise-4/videos_by_tag_year.csv' 
      WITH HEADER=true;

- Selezionare i video con tag `trailer` e anno `2015`

      SELECT *
      FROM videos_by_tag_year
      WHERE tag= 'trailer' AND added_year = 2015; 

- Selezionare i video con tag `cql` prima del `2015`

      SELECT *
      FROM videos_by_tag_year
      WHERE tag='cql' AND added_year < 2015; 

- Selezionare i video prima del `2015`

      SELECT * FROM videos_by_tag_year
      WHERE added_year < 2015
      ALLOW FILTERING;

- Selezionare i video con tag `cql` 

      SELECT *
      FROM videos_by_tag_year
      WHERE tag='cql'; 


### Exercise 4

- Troncare la tabella `videos`

      TRUNCATE videos;
      SELECT * FROM VIDEOS;

- Estendere la tabella originale `videos` con una colonna `tags` come collezione di stringhe

      ALTER TABLE videos ADD tags SET<TEXT>;

- Ripopolare `videos` caricando i dati che comprendono i tag. Fare riferimento al file `/root/labwork/exercise-5/videos_encoding.csv`

      COPY videos FROM '/root/labwork/exercise-5/videos.csv' WITH HEADER=true;

- Estendere `videos` con una colonna una colonna `video_encoding` di tipo UDT e così composta

  | Field name | Data type |
  | ---------- | --------- | 
  | bit_rates  | set\<text\> |
  | encoding   | text      |
  | height     | int       |
  | width      | int       |

      CREATE TYPE video_encoding ( 
            bit_rates SET<TEXT>, 
            encoding text, 
            height int, 
            width int
      ); 

      ALTER TABLE videos ADD encoding FROZEN<video_encoding>;

- Utilizzare sempre il commando copy per popolare solamente i valori della nuova colonna

      COPY videos (video_id, encoding)
      FROM '/root/labwork/exercise-5/videos_encoding.csv' 
      WITH HEADER=true;

- Selezionare 10 video

      SELECT * FROM videos LIMIT 10;


### Exercise 5

- Verificare le colonne presenti nel file .cql

- Creare una tabella `videos_count_by_tag` con un contatore che conteggi il numero di video a cui è assegnato un determinato tag

      CREATE TABLE videos_count_by_tag(  
            tag TEXT, 
            added_year int,
            video_count COUNTER,
            PRIMARY KEY ((tag), added_year) 
      );

- Eseguire lo script `/root/labwork/exercise-6/videos_count_by_tag.cql`

      SOURCE '/root/labwork/exercise-6/videos_count_by_tag.cql';

- Selezionare 5 video

      SELECT * 
      FROM videos_count_by_tag 
      LIMIT 5;

- Selezionare il tag `You Are Awesome`

      SELECT *
      FROM videos_count_by_tag
      WHERE tag = 'You Are Awesome';

- Aumentare di 10 il `video_count` per il tag `You Are Awesome` e anno `2015`

      UPDATE videos_count_by_tag 
      SET video_count = video_count + 10
      WHERE tag = 'You Are Awesome' AND added_year = 2015;

- Selezionare il tag `You Are Awesome`

      SELECT *
      FROM videos_count_by_tag
      WHERE tag = 'You Are Awesome';
