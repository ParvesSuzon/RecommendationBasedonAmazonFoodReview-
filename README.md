**Import Necessary Libraries**
```
import numpy as np
import pandas as pd
import math
import json
import time
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.metrics.pairwise import cosine_similarity
from sklearn.model_selection import train_test_split
from sklearn.neighbors import NearestNeighbors
#from sklearn.externals import joblib
import scipy.sparse
from scipy.sparse import csr_matrix
import warnings; warnings.simplefilter('ignore')
%matplotlib inline

# Input data files are available in the "../input/" directory.
# For example, running this (by clicking run or pressing Shift+Enter) will list all files under the input directory

import os
for dirname, _, filenames in os.walk('/Downloads/input'):
    for filename in filenames:
        print(os.path.join(dirname, filename))

# Any results you write to the current directory are saved as output.
```
**Importing the Dataset**
```
#Import the data set
df = pd.read_csv('/kaggle/input/amazon-fine-food-reviews/Reviews.csv')
```

```
df.head()
```

Split the data randomly into train and test dataset. ( For example split it in 70/30 ratio)
```
#Split the training and test data in the ratio 70:30
train_data, test_data = train_test_split(df_final, test_size = 0.3, random_state=0)

print(train_data.head(5))
```

Output
```
         ProductId          UserId  Score
399863  B002IEVJRY  A1N5FSCYN4796F      3
20262   B001BDDTB2  A1Q7A78VSQ5GQ4      5
139611  B001BCXTGS  A2PNOU7NXB1JE4      3
455504  B005HG9ERW  A2SZLNSI5KOQJT      3
512008  B0028PDER6   ALSAOZ1V546VT      5
```
```
def shape():
    print("Test data shape: ", test_data.shape)
    print("Train data shape: ", train_data.shape)
shape()
```
Output
```
Test data shape:  (6883, 3)
Train data shape:  (16058, 3)
```

Build Collaborative Filtering model
Model-based Collaborative Filtering: Singular Value Decomposition
```
df_CF = pd.concat([train_data, test_data]).reset_index()
df_CF.tail()
```

Output
```
	    index 	ProductId 	UserId 	        Score
22936 	275741 	B001M23WVY 	AY1EF0GOH80EK 	2
22937 	281102 	B002R8SLUY 	A16AXQ11SZA8SQ 	5
22938 	205589 	B00473PVVO 	A281NPSIMI1C2R 	5
22939 	303238 	B0002DGRZC 	AJD41FBJD9010 	5
22940 	36703 	B000EEWZD2 	A2M9D9BDHONV3Y 	3
```

```
#User-based Collaborative Filtering
# Matrix with row per 'user' and column per 'item' 
pivot_df = pd.pivot_table(df_CF,index=['UserId'], columns = 'ProductId', values = "Score")
pivot_df.fillna(0,inplace=True)
print(pivot_df.shape)
pivot_df.head()
```
Output
```
(267, 11313)
```
```
# Average PREDICTED rating for each item
preds_df.mean().head()
```
Output
```
ProductId
7310172001    0.001174
7310172101    0.001174
7800648702    0.004557
B00004CI84    0.039487
B00004CXX9    0.039487
dtype: float64
```

```
rmse_df = pd.concat([final_ratings_matrix.mean(), preds_df.mean()], axis=1)
rmse_df.columns = ['Avg_actual_ratings', 'Avg_predicted_ratings']
print(rmse_df.shape)
rmse_df['item_index'] = np.arange(0, rmse_df.shape[0], 1)
rmse_df.head()
```

```
(11313, 2)
```
```
RMSE = round((((rmse_df.Avg_actual_ratings - rmse_df.Avg_predicted_ratings) ** 2).mean() ** 0.5), 5)
print('\nRMSE SVD Model = {} \n'.format(RMSE))
```
Output
```
RMSE SVD Model = 0.00995 
```

Get top - K ( K = 5) recommendations. Since our goal is to recommend new products to each user based on his/her habits, we will recommend 5 new products.

```
# Enter 'userID' and 'num_recommendations' for the user #
userID = 200
num_recommendations = 5
recommend_items(userID, pivot_df, preds_df, num_recommendations)
```
```
Below are the recommended items for user(user_id = 200):

                   user_ratings  user_predictions
Recommended Items                                
B004BKLHOS                  0.0          0.823791
B0061IUIDY                  0.0          0.622365
B004JRO1S2                  0.0          0.538305
B0061IUKDM                  0.0          0.534249
B000EQT77M                  0.0          0.529929
```


Conclusion

Model-based Collaborative Filtering is a personalised recommender system, the recommendations are based on the past behavior of the user and it is not dependent on any additional information.

The Popularity-based recommender system is non-personalised and the recommendations are based on frequecy counts, which may be not suitable to the user.You can see the differance above for the user id 121 & 200, The Popularity based model has recommended the same set of 5 products to both but Collaborative Filtering based model has recommended entire different list based on the user past purchase history





