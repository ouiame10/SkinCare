 II. Prétraitement des données
=============================

2.1 Importation et traitement des données
-----------------------------------------

Ce script est utilisé pour automatiser le nettoyage des fichiers CSV dans un dossier. Il permet de préparer les données brutes à l'analyse en effectuant plusieurs opérations de nettoyage et de transformation. Chaque fichier nettoyé est sauvegardé dans un dossier de sortie.

2.1.1 Étapes principales
------------------------

- Charger les fichiers CSV depuis un dossier spécifié.
- Supprimer les colonnes contenant plus de 30% de valeurs manquantes.
- Remplacer les valeurs manquantes :
- Colonnes numériques : remplacer par la médiane.
- Colonnes non numériques : remplacer par "Inconnu".
- Supprimer les doublons pour éviter les répétitions.
- Supprimer les colonnes inutiles comme `id` (si présentes).
- Sauvegarder les fichiers nettoyés dans un dossier spécifique.

2.1.2 Installer les bibliothèques nécessaires
---------------------------------------------

Assurez-vous que les bibliothèques nécessaires sont installées avant d'exécuter le script.

.. code-block:: python

    import os
    import pandas as pd
    import numpy as np

2.1.3 Code principal
--------------------

Le code ci-dessous effectue les étapes mentionnées ci-dessus :

.. code-block:: python

    # Spécifiez le dossier contenant les fichiers CSV et le dossier de sortie
    input_folder = "/content/drive/MyDrive/DATASETS/dataset"
    output_folder = "/content/drive/MyDrive/DATASETS/cleaned"

    # Créez le dossier de sortie s'il n'existe pas
    os.makedirs(output_folder, exist_ok=True)

    # Parcourir tous les fichiers dans le dossier
    for filename in os.listdir(input_folder):
        if filename.endswith(".csv"):  # Vérifiez si le fichier est un CSV
            file_path = os.path.join(input_folder, filename)
            try:
                # Charger le CSV
                df = pd.read_csv(file_path)
                # Étapes de nettoyage (voir explications ci-dessus)
                ...

2.1.4 Résultats et sortie
-------------------------

- Les fichiers nettoyés sont sauvegardés dans : `/content/drive/MyDrive/DATASETS/cleaned`.
- Chaque fichier est prétraité pour garantir des données propres et prêtes à l'analyse.
6. Conversion d'un fichier CSV en un catalogue PDF
--------------------------------------------------

2.2 Importation et traitement des données

Ce script permet de transformer un fichier CSV contenant des informations sur des produits en un fichier PDF structuré et lisible. Il est utile pour générer des catalogues de produits ou des rapports détaillés. Les étapes principales incluent :

- **Charement des données CSV :** Lecture du fichier CSV contenant les informations des produits.
- **Création du fichier PDF :** Utilisation de la bibliothèque `FPDF` pour créer un fichier PDF structuré.
- **Ajout des données produit :** Parcours des lignes du DataFrame et ajout des informations (marque, nom, type, ingrédients, etc.) dans le fichier PDF.
- **Sauvegarde du PDF :** Enregistrement du fichier généré à l'emplacement spécifié.

Le fichier PDF résultant présente les produits de manière claire et organisée, adapté pour une consultation ou un partage.

**Chemin de sauvegarde par défaut :** `/content/drive/MyDrive/skinbot/primoData/datasheet_pdf.pdf`

.. code-block:: python

    import pandas as pd
    from fpdf import FPDF

    # Charger le fichier CSV (remplacez ce chemin par celui de votre fichier)
    file_path = '/content/drive/MyDrive/skinbot/primoData/cleaned_datasheet_modifie.csv'
    df = pd.read_csv(file_path)

    # Créer une instance de FPDF
    pdf = FPDF()
    pdf.set_auto_page_break(auto=True, margin=15)
    pdf.add_page()

    # Ajouter un titre
    pdf.set_font("Arial", 'B', 16)
    pdf.cell(200, 10, txt="Catalogue de Produits", ln=True, align='C')

    # Ajouter un saut de ligne
    pdf.ln(10)

    # Paramétrer la police pour les informations des produits
    pdf.set_font("Arial", size=12)

    # Parcourir chaque ligne du DataFrame et ajouter les informations dans le PDF
    for index, row in df.iterrows():
        # Encode the text fields to UTF-8 and then decode to latin-1, replacing unmappable characters
        brand = row['brand'].encode('utf-8', 'replace').decode('latin-1')
        name = row['name'].encode('utf-8', 'replace').decode('latin-1')
        type_ = row['type'].encode('utf-8', 'replace').decode('latin-1')
        ingredients = row['ingridients'].encode('utf-8', 'replace').decode('latin-1')
        after_use = row['afterUse'].encode('utf-8', 'replace').decode('latin-1')

        # Ajouter les informations au PDF
        pdf.cell(200, 10, txt=f"Marque : {brand}", ln=True)
        pdf.cell(200, 10, txt=f"Nom : {name}", ln=True)
        pdf.cell(200, 10, txt=f"Type : {type_}", ln=True)
        pdf.multi_cell(0, 10, txt=f"Ingrédients : {ingredients}")
        pdf.multi_cell(0, 10, txt=f"Utilisation / Problèmes de peau : {after_use}")
        pdf.ln(10)

    # Sauvegarder le fichier PDF
    pdf_output_path = '/content/drive/MyDrive/skinbot/primoData/datasheet_pdf.pdf'
    pdf.output(pdf_output_path)

    print(f"Le fichier PDF a été généré et sauvegardé à : {pdf_output_path}")

