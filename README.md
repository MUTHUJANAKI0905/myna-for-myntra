def load_and_clean_data(file_path):
    """
    Loads data from a CSV file and performs common cleaning operations.
    
    Args:
        file_path (str): The path to the data file (e.g., 'data.csv').
    
    Returns:
        pd.DataFrame: A cleaned Pandas DataFrame.
    """
    
    print(f"--- 1. Loading Data from: {file_path} ---")
    
    try:
        # Load the CSV file into a DataFrame
        df = pd.read_csv(file_path)
    except FileNotFoundError:
        print(f"Error: File not found at {file_path}")
        return None

    #2. Initial Data Inspection
    print("\n--- Initial Inspection (First 5 Rows) ---")
    print(df.head())
    
    print("\n--- Data Information (Data Types and Missing Values) ---")
    df.info()

    -----3. Cleaning Steps ---
    
    #  Handling Duplicates
    initial_rows = len(df)
    df.drop_duplicates(inplace=True)
    rows_dropped = initial_rows - len(df)
    print(f"\n--- Cleaning: Duplicates ---")
    print(f"Dropped {rows_dropped} duplicate rows.")

    # Standardizing Column Names 
    # Converts names like 'Product ID' to 'product_id'
    df.columns = df.columns.str.lower().str.replace(' ', '_')
    print("\n--- Cleaning: Column Names ---")
    print(f"New Columns: {list(df.columns)}")
    
    #  Handling Missing Values (Nulls/NaNs)
    print("\n--- Cleaning: Missing Values ---")
    
    # Check for columns with missing data
    missing_counts = df.isnull().sum()
    missing_cols = missing_counts[missing_counts > 0]
    
    if not missing_cols.empty:
        print("Missing values found in the following columns:")
        print(missing_cols)
        
       
        
        #  Fill missing values
        for col in missing_cols.index:
            # If the column is numeric, fill with the mean/median
            if pd.api.types.is_numeric_dtype(df[col]):
                median_val = df[col].median()
                df[col].fillna(median_val, inplace=True)
                print(f"- Filled missing values in '{col}' with the median ({median_val}).")
            
            # If the column is categorical, fill with the most frequenT
            elif pd.api.types.is_object_dtype(df[col]):
                df[col].fillna('UNKNOWN', inplace=True)
                print(f"- Filled missing values in '{col}' with 'UNKNOWN'.")
    else:
        print("No missing values found.")

    

    print("\n--- Final Cleaned Data Info ---")
    df.info()
    
    return df




dummy_data = {
    'Transaction ID': [1, 2, 3, 4, 5, 1],
    'Item Category': ['Food', 'Tech', 'Food', np.nan, 'Tech', 'Food'],
    'Price': [10.5, 99.9, np.nan, 5.0, 150.0, 10.5],
    'Quantity': [1, 2, 1, 3, 1, 1]
}
dummy_df = pd.DataFrame(dummy_data)
dummy_df.to_csv('sales_data.csv', index=False)

# Call the function with your file path
cleaned_data = load_and_clean_data('sales_data.csv')

if cleaned_data is not None:
    print("\n✅ Data Cleaning Complete. Ready for Analysis!")
    import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns



if 'cleaned_data' not in locals() or cleaned_data is None:
    print("Error: Please make sure 'cleaned_data' DataFrame is defined and loaded.")
else:
    print("--- Starting Exploratory Data Analysis (EDA) ---")

    
    # 1. Descriptive Statistics (Numerical Columns)

    
    print("\n--- 1. Descriptive Statistics ---")
    # Provides count, mean, std, min, max, and quartiles for numeric columns
    print(cleaned_data.describe())

    
    # 2. Value Counts (Categorical Columns)
    

    print("\n--- 2. Value Counts for Categorical Columns ---")
    
    # Iterate through object (string/categorical) columns
    for col in cleaned_data.select_dtypes(include='object').columns:
        print(f"\nValue Counts for '{col}':")
        # Shows the frequency of each unique value in the column
        print(cleaned_data[col].value_counts())

    
    # 3. Data Visualization (Histograms and Bar Plots)
    

    # Set visualization style
    sns.set_style("whitegrid")
    
    # Create figures for visualizations
    fig, axes = plt.subplots(1, 2, figsize=(14, 5))
    
    
    
   
    if 'price' in cleaned_data.columns and pd.api.types.is_numeric_dtype(cleaned_data['price']):
        sns.histplot(cleaned_data['price'], bins=10, kde=True, ax=axes[0], color='skyblue')
        axes[0].set_title('Distribution of Price')
        axes[0].set_xlabel('Price')
        axes[0].set_ylabel('Frequency')
    else:
        axes[0].text(0.5, 0.5, "Price column not found or not numeric.", 
                     horizontalalignment='center', verticalalignment='center')


    # --- Visualization B: Categorical Variable Counts ---
    
    
    if 'item_category' in cleaned_data.columns and pd.api.types.is_object_dtype(cleaned_data['item_category']):
        sns.countplot(x='item_category', data=cleaned_data, ax=axes[1], palette='viridis')
        axes[1].set_title('Count of Items by Category')
        axes[1].set_xlabel('Item Category')
        axes[1].set_ylabel('Count')
        axes[1].tick_params(axis='x', rotation=45)
    else:
        axes[1].text(0.5, 0.5, "Item Category column not found or not categorical.", 
                     horizontalalignment='center', verticalalignment='center')

    plt.tight_layout()
    plt.show()

    print("\n✅ EDA Complete. ")
