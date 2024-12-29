III. Ingestion dans une base vectorielle:
==================================================


Cce module couvre les étapes de préparation et d’ingestion des documents pour les rendre accessibles dans une base vectorielle. Il fait partie de la phase de **Récupération** (Retrieval) dans le processus RAG.

.. attention::
   - Assurez-vous que le répertoire contenant les fichiers PDF est bien configuré avant de lancer ce script.

.. code-block:: python

   import os
   import pdfplumber
   from langchain_chroma import Chroma
   from langchain_ollama import OllamaEmbeddings
   from langchain.text_splitter import CharacterTextSplitter
   from langchain.schema import Document

   pdf_directory = "C:/Users/PC-18/Desktop/skinbot"

3.1 Extraction des Documents PDF
--------------------------------

Cette étape lit le contenu des fichiers PDF et retourne un texte structuré.

.. code-block:: python

   def read_pdf(file_path):
       """Lit un fichier PDF et retourne son contenu sous forme de texte."""
       try:
           with pdfplumber.open(file_path) as pdf:
               text = ""
               for page_num, page in enumerate(pdf.pages):
                   try:
                       page_text = page.extract_text()
                       if page_text:
                           text += page_text + "\n"
                       print(f"Page {page_num + 1}/{len(pdf.pages)} extraite avec succès.")
                   except Exception as e:
                       print(f"Erreur lors de l'extraction de la page {page_num + 1}: {e}")
               return text
       except Exception as e:
           print(f"Erreur lors de l'ouverture du fichier PDF {file_path}: {e}")
           return ""

3.2 Chargement des Documents
----------------------------

Charge tous les fichiers PDF d’un répertoire et les convertit en objets `Document`.

.. code-block:: python

   def load_documents_from_directory(directory_path):
       """Charge les documents PDF depuis un répertoire."""
       files = [os.path.join(directory_path, file) for file in os.listdir(directory_path) if file.endswith(".pdf")]
       documents = []
       for file in files:
           print(f"Traitement du fichier : {file}")
           content = read_pdf(file)
           if content.strip():
               documents.append(Document(page_content=content))
           else:
               print(f"Aucun contenu extrait du fichier {file}.")
       return documents

3.3 Segmentation et Ingestion
-----------------------------

Cette étape segmente les textes en chunks optimisés pour une base vectorielle et les ingère dans une base Chroma.

.. code-block:: python

   def ingest_into_vector_store(combined_texts):
       """Segmente et ingère les documents dans une base vectorielle."""
       text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
           chunk_size=2000,
           chunk_overlap=200,
           separator=".",
           keep_separator=False
       )
       doc_splits = text_splitter.split_documents(
           [Document(page_content=text.replace(". ", ".\n")) for text in combined_texts]
       )

       db = Chroma(
           persist_directory="C:/Users/PC-18/Desktop/skinbot/finalbv",
           embedding_function=OllamaEmbeddings(model="mxbai-embed-large:latest"),
           collection_name="rag-chroma"
       )
       db.add_documents(doc_splits)
       print("Données ingérées avec succès dans la base vectorielle.")

3.4 Initialisation de la Base Vectorielle
-----------------------------------------

Pour interroger ou étendre les données déjà ingérées, vous pouvez initialiser la base vectorielle.

.. code-block:: python

   def initialize_vector_store():
       """Initialise la base vectorielle pour la récupération."""
       return Chroma(
           persist_directory="C:/Users/PC-18/Desktop/skinbot/finalbv",
           embedding_function=OllamaEmbeddings(model="mxbai-embed-large:latest"),
           collection_name="rag-chroma"
       )

-------------------------
