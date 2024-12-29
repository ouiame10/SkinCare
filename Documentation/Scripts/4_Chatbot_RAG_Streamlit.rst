5. Implémentation d'un Chatbot RAG avec Streamlit
=================================================

Ce module implémente un **Chatbot RAG** (Retrieval-Augmented Generation) utilisant **Streamlit** comme interface utilisateur. Le chatbot peut répondre aux questions des utilisateurs en s’appuyant sur des fichiers PDF téléchargés ou une base de vecteurs existante. Ce script couvre les étapes de **Récupération (Retrieval)** et **Génération (Generation)** du processus RAG.

5.1 Lecture des Documents PDF
-----------------------------

Cette partie du code utilise `PyPDF2` pour extraire le texte d’un fichier PDF téléchargé par l'utilisateur.

.. code-block:: python

   def read_pdf(file):
       """Lit un fichier PDF et retourne son contenu sous forme de texte."""
       pdfReader = PyPDF2.PdfReader(file)
       all_page_text = ""
       for page in pdfReader.pages:
           all_page_text += page.extract_text() + "\n"
       return all_page_text

**Rôle** :
- Extraire le contenu texte des documents PDF pour les rendre exploitables dans le pipeline RAG.

---

5.2 Recherche dans la Base Vectorielle
--------------------------------------

La fonction `retrieve_from_db` interroge une base vectorielle existante pour trouver les informations pertinentes en fonction de la question posée.

.. code-block:: python

   def retrieve_from_db(question):
       """Recherche dans la base vectorielle et génère une réponse basée sur le contexte."""
       model = ChatOllama(model="mistral")
       db = initialize_vector_store()
       retriever = db.similarity_search(question, k=2)

       after_rag_template = """Answer the question based only on the following context:
       {context}
       Question: {question}
       if there is no answer, please answer with "I m sorry, the context is not enough to answer the question."
       """

       after_rag_prompt = ChatPromptTemplate.from_template(after_rag_template)
       after_rag_chain = (
           {"context": RunnablePassthrough(), "question": RunnablePassthrough()}
           | after_rag_prompt
           | model
           | StrOutputParser()
       )
       return after_rag_chain.invoke({"context": retriever, "question": question})

**Rôle** :
- Effectuer une recherche basée sur la similarité pour trouver les données pertinentes.
- Générer une réponse basée uniquement sur le contexte récupéré.

---

5.3 Récupération et Segmentation Locale
---------------------------------------

La fonction `retriever` permet de segmenter les documents téléchargés localement en chunks avant de les intégrer dans une base vectorielle temporaire pour effectuer une recherche.

.. code-block:: python

   def retriever(doc, question):
       """Effectue une récupération basée sur des documents segmentés localement."""
       model_local = ChatOllama(model="mistral")
       doc = Document(page_content=doc)
       doc = [doc]
       text_splitter = CharacterTextSplitter.from_tiktoken_encoder(chunk_size=800, chunk_overlap=0)
       doc_splits = text_splitter.split_documents(doc)

       vectorstore = Chroma.from_documents(
           documents=doc_splits,
           collection_name="rag-chroma",
           embedding=OllamaEmbeddings(model="mxbai-embed-large:latest"),
       )
       retriever = vectorstore.as_retriever(k=2)

       after_rag_template = """Answer the question based only on the following context:
       {context}
       Question: {question}
       if there is no answer, please answer with "I m sorry, the context is not enough to answer the question."
       """
       after_rag_prompt = ChatPromptTemplate.from_template(after_rag_template)
       after_rag_chain = (
           {"context": retriever, "question": RunnablePassthrough()}
           | after_rag_prompt
           | model_local
           | StrOutputParser()
       )
       return after_rag_chain.invoke(question)

**Rôle** :
- Préparer les documents en segments optimisés pour la recherche.
- Permettre une recherche locale dans des données non persistantes.

---

5.4 Interface Utilisateur avec Streamlit
----------------------------------------

Cette section configure l'interface utilisateur via **Streamlit**. Elle permet aux utilisateurs de télécharger un fichier PDF ou de poser une question directement.

.. code-block:: python

   st.title("RAG Chatbot")
   st.write("This is a RAG chatbot that can answer questions based on a given context.")
   file = st.file_uploader("Upload a PDF file", type=["pdf"])
   if file:
       doc = read_pdf(file)
       question = st.text_input("Ask a question")
       if st.button("Ask"):
           answer = retriever(doc, question)
           st.write(answer)
   else:
       question = st.text_input("Ask a question")
       if st.button("Ask"):
           answer = retrieve_from_db(question)
           st.write(answer)

**Rôle** :
- Fournir une interface interactive pour les utilisateurs.
- Intégrer les étapes de récupération et de génération en fonction des données fournies.

---

### Résumé

Ce code couvre les étapes suivantes du pipeline RAG :
1. **Récupération (Retrieval)** : Extraction des données pertinentes depuis une base vectorielle ou des documents téléchargés.
2. **Génération (Generation)** : Création de réponses basées sur le contexte récupéré.

Il utilise **Streamlit** pour offrir une expérience utilisateur fluide .
