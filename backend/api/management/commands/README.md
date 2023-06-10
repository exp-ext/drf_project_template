<h1 align="center">Packer</h1>

<p align="center">
    <img src="https://github.com/exp-ext/drf_project_template/blob/main/backend/static/git/big.jpg" width="400">
</p>


<h2>Data Conversion Script</h2>

<p>This script is designed to convert data from various file formats into corresponding Django model instances and store them in the database. The script uses the provided <code>MODEL_DATA</code> dictionary to define the conversion settings for each model.</p>

<h3>Prerequisites</h3>

<p>Before running the script, make sure you have the following dependencies installed:</p>

<ul>
<li><code>pandas</code>: A data manipulation library used for reading and cleaning the data files.</li>
<li>Django: The web framework used for the database models.</li>
</ul>

<h3>Setup</h3>

<ol>

<li>
<p>Import the necessary modules and packages:</p><pre>

```python
from pathlib import Path
import pandas as pd
from api.models import CargotypeInfo, Carton, CartonPrice, Sku, SkuCargotypes

```

</li>

<li>
<p>Define the path where the data files are located:</p>

```python
HOME_PATH = f'{Path.home()}/Desktop/data'
```

</li>

<li>

<p>Configure the <code>MODEL_DATA</code> dictionary:</p>

<ul>

<li>
<p>Each key represents a Django model.</p>
</li>

<li>
<p>The values are dictionaries containing the conversion settings for each model.</p>
<ul>
    <li><code>type</code>: The Pandas function to use for reading the file (e.g., <code>pd.read_csv</code>, <code>pd.read_excel</code>).</li>
    <li><code>file_name</code>: The name of the data file.</li><li><code>model_fields</code>: The fields of the model to populate with the converted data.</li>
    <li><code>file_fields</code>: The corresponding fields in the data file.</li>
    <li><code>cleaner</code>: A list of functions to clean the data values before assigning them to the model fields.</li>
    <li><code>getter</code>: A list of getter functions to retrieve related data if needed.</li>
</ul>
</li>

</ul>

```python
MODEL_DATA = {
    Carton: {
        'type': pd.read_csv,
        'file_name': 'carton.csv',
        'model_fields': [
            'cartontype', 'length', 'width', 'height', 'displayrfpack'
        ],
        'file_fields': [
            'CARTONTYPE', 'LENGTH', 'WIDTH', 'HEIGHT', 'DISPLAYRFPACK'
        ],
        'cleaner': [
            str, float, float, float, bool
        ],
        'getter': [
            None, None, None, None, None
        ],
    },
    # Add configuration settings for other models...
}
```

</li>

</ol>

<h3>Data Conversion</h3>

<p>The <code>convert_to_models()</code> function is responsible for converting the data files into model instances and storing them in the database.</p>

```python
def convert_to_models():
    """
    The function reads the files for each model in MODEL_DATA, cleans the data
    using the cleaner function, retrieves related data using the getter
    function, creates model instances using the cleaned data, and bulk creates
    them in the database.

    If any error occurs during the process, it is added to the result list
    and returned at the end.
    """
    result = []
    for model, model_data in MODEL_DATA.items():
        try:
            data = model_data['type'](f'{HOME_PATH}/{model_data["file_name"]}')
            clean_data = []
            for _, row in data.iterrows():
                clean_row = model(**{
                    model_fields: (
                        getter[0].objects.get_or_create(**{getter[1]: row[file_field]})[0]
                        if getter else cleaner(row[file_field])
                    )
                    for model_fields, file_field, cleaner, getter in zip(
                        model_data['model_fields'],
                        model_data['file_fields'],
                        model_data['cleaner'],
                        model_data['getter']
                    )
                })
                clean_data.append(clean_row)
            model.objects.bulk_create(clean_data)
        except Exception as error:
            result.extend(['error', f'{model}: {error}'])
    return result
```


<p>To run the data conversion process, simply call the <code>convert_to_models()</code> function.</p>

```python
results = convert_to_models()
```

<p>Any errors that occur during the conversion process will be added to the <code>results</code> list and returned at the end.</p>
<p>Make sure you have a database connection configured in your Django project before running this script.</p>
