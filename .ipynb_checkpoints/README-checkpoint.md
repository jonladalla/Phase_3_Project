{
 "cells": [
  {
   "cell_type": "markdown",
   "id": "bcee8db8",
   "metadata": {},
   "source": [
    "# Telecom Churn Rate Predictions"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d9182c0c",
   "metadata": {},
   "source": [
    "## The Problem\n",
    "\n",
    "Our goal is to predict if an individual customer will drop Telecom or not. We'll use machine learning models trained from 80% of the sample data and the remaining 20% will be applied to assess the trained models accuracy in predicting whether or not the customers will churn. "
   ]
  },
  {
   "cell_type": "markdown",
   "id": "dac963cc",
   "metadata": {},
   "source": [
    "## Collecting Data\n",
    "\n",
    "This data set came from Kaggle sourced from the IBM sample data collection (https://www.kaggle.com/becksddf/churn-in-telecoms-dataset). We'll need to use certain libraries to accomplish our goal moving forward. I'll import Pandas, Numpy, Matplotlib and Seaborn initially. "
   ]
  },
  {
   "cell_type": "markdown",
   "id": "649242e1",
   "metadata": {},
   "source": [
    "#### Importing Libraries"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "id": "0755dd17",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "        <script type=\"text/javascript\">\n",
       "        window.PlotlyConfig = {MathJaxConfig: 'local'};\n",
       "        if (window.MathJax) {MathJax.Hub.Config({SVG: {font: \"STIX-Web\"}});}\n",
       "        if (typeof require !== 'undefined') {\n",
       "        require.undef(\"plotly\");\n",
       "        requirejs.config({\n",
       "            paths: {\n",
       "                'plotly': ['https://cdn.plot.ly/plotly-2.9.0.min']\n",
       "            }\n",
       "        });\n",
       "        require(['plotly'], function(Plotly) {\n",
       "            window._Plotly = Plotly;\n",
       "        });\n",
       "        }\n",
       "        </script>\n",
       "        "
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# scientific computing libaries\n",
    "import pandas as pd\n",
    "import numpy as np\n",
    "\n",
    "# data mining libaries\n",
    "from sklearn.tree import DecisionTreeClassifier\n",
    "from sklearn.ensemble import RandomForestClassifier\n",
    "from sklearn.preprocessing import LabelEncoder\n",
    "from sklearn.decomposition import PCA#, FastICA\n",
    "from sklearn.model_selection import train_test_split, KFold, StratifiedKFold, GridSearchCV, learning_curve\n",
    "from sklearn import svm\n",
    "from sklearn.linear_model import LogisticRegression\n",
    "from sklearn.neighbors import KNeighborsClassifier\n",
    "from sklearn.metrics import roc_curve, auc, confusion_matrix, accuracy_score, f1_score, precision_score, recall_score, roc_auc_score\n",
    "\n",
    "from imblearn.pipeline import make_pipeline, Pipeline\n",
    "from imblearn.over_sampling import SMOTE\n",
    "\n",
    "#plot libaries\n",
    "import plotly\n",
    "import plotly.graph_objs as go\n",
    "import plotly.figure_factory as ff\n",
    "from plotly.offline import init_notebook_mode\n",
    "init_notebook_mode(connected=True) # to show plots in notebook\n",
    "from matplotlib import pyplot as plt\n",
    "%matplotlib inline\n",
    "import seaborn as sns\n",
    "\n",
    "# offline plotly\n",
    "from plotly.offline import plot, iplot\n",
    "\n",
    "# do not show any warnings\n",
    "import warnings\n",
    "warnings.filterwarnings('ignore')\n",
    "\n",
    "SEED = 17 # specify seed for reproducable results\n",
    "pd.set_option('display.max_columns', None) # prevents abbreviation (with '...') of columns in prints\n",
    "\n",
    "import os\n",
    "for dirname, _, filenames in os.walk('/kaggle/input'):\n",
    "    for filename in filenames:\n",
    "        print(os.path.join(dirname, filename))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "59949e04",
   "metadata": {},
   "source": [
    "#### Loading the data frame"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "id": "5576f54a",
   "metadata": {},
   "outputs": [],
   "source": [
    "df = pd.read_csv('Telcom.csv')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "ed069d43",
   "metadata": {},
   "source": [
    "# 2. Data Exploration\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "id": "3399a135",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>state</th>\n",
       "      <th>account length</th>\n",
       "      <th>area code</th>\n",
       "      <th>phone number</th>\n",
       "      <th>international plan</th>\n",
       "      <th>voice mail plan</th>\n",
       "      <th>number vmail messages</th>\n",
       "      <th>total day minutes</th>\n",
       "      <th>total day calls</th>\n",
       "      <th>total day charge</th>\n",
       "      <th>total eve minutes</th>\n",
       "      <th>total eve calls</th>\n",
       "      <th>total eve charge</th>\n",
       "      <th>total night minutes</th>\n",
       "      <th>total night calls</th>\n",
       "      <th>total night charge</th>\n",
       "      <th>total intl minutes</th>\n",
       "      <th>total intl calls</th>\n",
       "      <th>total intl charge</th>\n",
       "      <th>customer service calls</th>\n",
       "      <th>churn</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>KS</td>\n",
       "      <td>128</td>\n",
       "      <td>415</td>\n",
       "      <td>382-4657</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>25</td>\n",
       "      <td>265.1</td>\n",
       "      <td>110</td>\n",
       "      <td>45.07</td>\n",
       "      <td>197.4</td>\n",
       "      <td>99</td>\n",
       "      <td>16.78</td>\n",
       "      <td>244.7</td>\n",
       "      <td>91</td>\n",
       "      <td>11.01</td>\n",
       "      <td>10.0</td>\n",
       "      <td>3</td>\n",
       "      <td>2.70</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>OH</td>\n",
       "      <td>107</td>\n",
       "      <td>415</td>\n",
       "      <td>371-7191</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>26</td>\n",
       "      <td>161.6</td>\n",
       "      <td>123</td>\n",
       "      <td>27.47</td>\n",
       "      <td>195.5</td>\n",
       "      <td>103</td>\n",
       "      <td>16.62</td>\n",
       "      <td>254.4</td>\n",
       "      <td>103</td>\n",
       "      <td>11.45</td>\n",
       "      <td>13.7</td>\n",
       "      <td>3</td>\n",
       "      <td>3.70</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>NJ</td>\n",
       "      <td>137</td>\n",
       "      <td>415</td>\n",
       "      <td>358-1921</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>243.4</td>\n",
       "      <td>114</td>\n",
       "      <td>41.38</td>\n",
       "      <td>121.2</td>\n",
       "      <td>110</td>\n",
       "      <td>10.30</td>\n",
       "      <td>162.6</td>\n",
       "      <td>104</td>\n",
       "      <td>7.32</td>\n",
       "      <td>12.2</td>\n",
       "      <td>5</td>\n",
       "      <td>3.29</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>OH</td>\n",
       "      <td>84</td>\n",
       "      <td>408</td>\n",
       "      <td>375-9999</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>299.4</td>\n",
       "      <td>71</td>\n",
       "      <td>50.90</td>\n",
       "      <td>61.9</td>\n",
       "      <td>88</td>\n",
       "      <td>5.26</td>\n",
       "      <td>196.9</td>\n",
       "      <td>89</td>\n",
       "      <td>8.86</td>\n",
       "      <td>6.6</td>\n",
       "      <td>7</td>\n",
       "      <td>1.78</td>\n",
       "      <td>2</td>\n",
       "      <td>False</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>OK</td>\n",
       "      <td>75</td>\n",
       "      <td>415</td>\n",
       "      <td>330-6626</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>166.7</td>\n",
       "      <td>113</td>\n",
       "      <td>28.34</td>\n",
       "      <td>148.3</td>\n",
       "      <td>122</td>\n",
       "      <td>12.61</td>\n",
       "      <td>186.9</td>\n",
       "      <td>121</td>\n",
       "      <td>8.41</td>\n",
       "      <td>10.1</td>\n",
       "      <td>3</td>\n",
       "      <td>2.73</td>\n",
       "      <td>3</td>\n",
       "      <td>False</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "  state  account length  area code phone number international plan  \\\n",
       "0    KS             128        415     382-4657                 no   \n",
       "1    OH             107        415     371-7191                 no   \n",
       "2    NJ             137        415     358-1921                 no   \n",
       "3    OH              84        408     375-9999                yes   \n",
       "4    OK              75        415     330-6626                yes   \n",
       "\n",
       "  voice mail plan  number vmail messages  total day minutes  total day calls  \\\n",
       "0             yes                     25              265.1              110   \n",
       "1             yes                     26              161.6              123   \n",
       "2              no                      0              243.4              114   \n",
       "3              no                      0              299.4               71   \n",
       "4              no                      0              166.7              113   \n",
       "\n",
       "   total day charge  total eve minutes  total eve calls  total eve charge  \\\n",
       "0             45.07              197.4               99             16.78   \n",
       "1             27.47              195.5              103             16.62   \n",
       "2             41.38              121.2              110             10.30   \n",
       "3             50.90               61.9               88              5.26   \n",
       "4             28.34              148.3              122             12.61   \n",
       "\n",
       "   total night minutes  total night calls  total night charge  \\\n",
       "0                244.7                 91               11.01   \n",
       "1                254.4                103               11.45   \n",
       "2                162.6                104                7.32   \n",
       "3                196.9                 89                8.86   \n",
       "4                186.9                121                8.41   \n",
       "\n",
       "   total intl minutes  total intl calls  total intl charge  \\\n",
       "0                10.0                 3               2.70   \n",
       "1                13.7                 3               3.70   \n",
       "2                12.2                 5               3.29   \n",
       "3                 6.6                 7               1.78   \n",
       "4                10.1                 3               2.73   \n",
       "\n",
       "   customer service calls  churn  \n",
       "0                       1  False  \n",
       "1                       1  False  \n",
       "2                       0  False  \n",
       "3                       2  False  \n",
       "4                       3  False  "
      ]
     },
     "execution_count": 3,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "id": "2aabbc33",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "<class 'pandas.core.frame.DataFrame'>\n",
      "RangeIndex: 3333 entries, 0 to 3332\n",
      "Data columns (total 21 columns):\n",
      " #   Column                  Non-Null Count  Dtype  \n",
      "---  ------                  --------------  -----  \n",
      " 0   state                   3333 non-null   object \n",
      " 1   account length          3333 non-null   int64  \n",
      " 2   area code               3333 non-null   int64  \n",
      " 3   phone number            3333 non-null   object \n",
      " 4   international plan      3333 non-null   object \n",
      " 5   voice mail plan         3333 non-null   object \n",
      " 6   number vmail messages   3333 non-null   int64  \n",
      " 7   total day minutes       3333 non-null   float64\n",
      " 8   total day calls         3333 non-null   int64  \n",
      " 9   total day charge        3333 non-null   float64\n",
      " 10  total eve minutes       3333 non-null   float64\n",
      " 11  total eve calls         3333 non-null   int64  \n",
      " 12  total eve charge        3333 non-null   float64\n",
      " 13  total night minutes     3333 non-null   float64\n",
      " 14  total night calls       3333 non-null   int64  \n",
      " 15  total night charge      3333 non-null   float64\n",
      " 16  total intl minutes      3333 non-null   float64\n",
      " 17  total intl calls        3333 non-null   int64  \n",
      " 18  total intl charge       3333 non-null   float64\n",
      " 19  customer service calls  3333 non-null   int64  \n",
      " 20  churn                   3333 non-null   bool   \n",
      "dtypes: bool(1), float64(8), int64(8), object(4)\n",
      "memory usage: 524.2+ KB\n"
     ]
    }
   ],
   "source": [
    "df.info()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "id": "ef9ebc85",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "['no' 'yes']\n",
      "['yes' 'no']\n",
      "['KS' 'OH' 'NJ' 'OK' 'AL' 'MA' 'MO' 'LA' 'WV' 'IN' 'RI' 'IA' 'MT' 'NY'\n",
      " 'ID' 'VT' 'VA' 'TX' 'FL' 'CO' 'AZ' 'SC' 'NE' 'WY' 'HI' 'IL' 'NH' 'GA'\n",
      " 'AK' 'MD' 'AR' 'WI' 'OR' 'MI' 'DE' 'UT' 'CA' 'MN' 'SD' 'NC' 'WA' 'NM'\n",
      " 'NV' 'DC' 'KY' 'ME' 'MS' 'TN' 'PA' 'CT' 'ND']\n"
     ]
    }
   ],
   "source": [
    "#checking the categories for object data types\n",
    "print(df['international plan'].unique())\n",
    "print(df['voice mail plan'].unique())\n",
    "print(df['state'].unique())"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "id": "3aee97b8",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>account length</th>\n",
       "      <th>area code</th>\n",
       "      <th>number vmail messages</th>\n",
       "      <th>total day minutes</th>\n",
       "      <th>total day calls</th>\n",
       "      <th>total day charge</th>\n",
       "      <th>total eve minutes</th>\n",
       "      <th>total eve calls</th>\n",
       "      <th>total eve charge</th>\n",
       "      <th>total night minutes</th>\n",
       "      <th>total night calls</th>\n",
       "      <th>total night charge</th>\n",
       "      <th>total intl minutes</th>\n",
       "      <th>total intl calls</th>\n",
       "      <th>total intl charge</th>\n",
       "      <th>customer service calls</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>count</th>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "      <td>3333.000000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>mean</th>\n",
       "      <td>101.064806</td>\n",
       "      <td>437.182418</td>\n",
       "      <td>8.099010</td>\n",
       "      <td>179.775098</td>\n",
       "      <td>100.435644</td>\n",
       "      <td>30.562307</td>\n",
       "      <td>200.980348</td>\n",
       "      <td>100.114311</td>\n",
       "      <td>17.083540</td>\n",
       "      <td>200.872037</td>\n",
       "      <td>100.107711</td>\n",
       "      <td>9.039325</td>\n",
       "      <td>10.237294</td>\n",
       "      <td>4.479448</td>\n",
       "      <td>2.764581</td>\n",
       "      <td>1.562856</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>std</th>\n",
       "      <td>39.822106</td>\n",
       "      <td>42.371290</td>\n",
       "      <td>13.688365</td>\n",
       "      <td>54.467389</td>\n",
       "      <td>20.069084</td>\n",
       "      <td>9.259435</td>\n",
       "      <td>50.713844</td>\n",
       "      <td>19.922625</td>\n",
       "      <td>4.310668</td>\n",
       "      <td>50.573847</td>\n",
       "      <td>19.568609</td>\n",
       "      <td>2.275873</td>\n",
       "      <td>2.791840</td>\n",
       "      <td>2.461214</td>\n",
       "      <td>0.753773</td>\n",
       "      <td>1.315491</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>min</th>\n",
       "      <td>1.000000</td>\n",
       "      <td>408.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>23.200000</td>\n",
       "      <td>33.000000</td>\n",
       "      <td>1.040000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>0.000000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>25%</th>\n",
       "      <td>74.000000</td>\n",
       "      <td>408.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>143.700000</td>\n",
       "      <td>87.000000</td>\n",
       "      <td>24.430000</td>\n",
       "      <td>166.600000</td>\n",
       "      <td>87.000000</td>\n",
       "      <td>14.160000</td>\n",
       "      <td>167.000000</td>\n",
       "      <td>87.000000</td>\n",
       "      <td>7.520000</td>\n",
       "      <td>8.500000</td>\n",
       "      <td>3.000000</td>\n",
       "      <td>2.300000</td>\n",
       "      <td>1.000000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>50%</th>\n",
       "      <td>101.000000</td>\n",
       "      <td>415.000000</td>\n",
       "      <td>0.000000</td>\n",
       "      <td>179.400000</td>\n",
       "      <td>101.000000</td>\n",
       "      <td>30.500000</td>\n",
       "      <td>201.400000</td>\n",
       "      <td>100.000000</td>\n",
       "      <td>17.120000</td>\n",
       "      <td>201.200000</td>\n",
       "      <td>100.000000</td>\n",
       "      <td>9.050000</td>\n",
       "      <td>10.300000</td>\n",
       "      <td>4.000000</td>\n",
       "      <td>2.780000</td>\n",
       "      <td>1.000000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>75%</th>\n",
       "      <td>127.000000</td>\n",
       "      <td>510.000000</td>\n",
       "      <td>20.000000</td>\n",
       "      <td>216.400000</td>\n",
       "      <td>114.000000</td>\n",
       "      <td>36.790000</td>\n",
       "      <td>235.300000</td>\n",
       "      <td>114.000000</td>\n",
       "      <td>20.000000</td>\n",
       "      <td>235.300000</td>\n",
       "      <td>113.000000</td>\n",
       "      <td>10.590000</td>\n",
       "      <td>12.100000</td>\n",
       "      <td>6.000000</td>\n",
       "      <td>3.270000</td>\n",
       "      <td>2.000000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>max</th>\n",
       "      <td>243.000000</td>\n",
       "      <td>510.000000</td>\n",
       "      <td>51.000000</td>\n",
       "      <td>350.800000</td>\n",
       "      <td>165.000000</td>\n",
       "      <td>59.640000</td>\n",
       "      <td>363.700000</td>\n",
       "      <td>170.000000</td>\n",
       "      <td>30.910000</td>\n",
       "      <td>395.000000</td>\n",
       "      <td>175.000000</td>\n",
       "      <td>17.770000</td>\n",
       "      <td>20.000000</td>\n",
       "      <td>20.000000</td>\n",
       "      <td>5.400000</td>\n",
       "      <td>9.000000</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "       account length    area code  number vmail messages  total day minutes  \\\n",
       "count     3333.000000  3333.000000            3333.000000        3333.000000   \n",
       "mean       101.064806   437.182418               8.099010         179.775098   \n",
       "std         39.822106    42.371290              13.688365          54.467389   \n",
       "min          1.000000   408.000000               0.000000           0.000000   \n",
       "25%         74.000000   408.000000               0.000000         143.700000   \n",
       "50%        101.000000   415.000000               0.000000         179.400000   \n",
       "75%        127.000000   510.000000              20.000000         216.400000   \n",
       "max        243.000000   510.000000              51.000000         350.800000   \n",
       "\n",
       "       total day calls  total day charge  total eve minutes  total eve calls  \\\n",
       "count      3333.000000       3333.000000        3333.000000      3333.000000   \n",
       "mean        100.435644         30.562307         200.980348       100.114311   \n",
       "std          20.069084          9.259435          50.713844        19.922625   \n",
       "min           0.000000          0.000000           0.000000         0.000000   \n",
       "25%          87.000000         24.430000         166.600000        87.000000   \n",
       "50%         101.000000         30.500000         201.400000       100.000000   \n",
       "75%         114.000000         36.790000         235.300000       114.000000   \n",
       "max         165.000000         59.640000         363.700000       170.000000   \n",
       "\n",
       "       total eve charge  total night minutes  total night calls  \\\n",
       "count       3333.000000          3333.000000        3333.000000   \n",
       "mean          17.083540           200.872037         100.107711   \n",
       "std            4.310668            50.573847          19.568609   \n",
       "min            0.000000            23.200000          33.000000   \n",
       "25%           14.160000           167.000000          87.000000   \n",
       "50%           17.120000           201.200000         100.000000   \n",
       "75%           20.000000           235.300000         113.000000   \n",
       "max           30.910000           395.000000         175.000000   \n",
       "\n",
       "       total night charge  total intl minutes  total intl calls  \\\n",
       "count         3333.000000         3333.000000       3333.000000   \n",
       "mean             9.039325           10.237294          4.479448   \n",
       "std              2.275873            2.791840          2.461214   \n",
       "min              1.040000            0.000000          0.000000   \n",
       "25%              7.520000            8.500000          3.000000   \n",
       "50%              9.050000           10.300000          4.000000   \n",
       "75%             10.590000           12.100000          6.000000   \n",
       "max             17.770000           20.000000         20.000000   \n",
       "\n",
       "       total intl charge  customer service calls  \n",
       "count        3333.000000             3333.000000  \n",
       "mean            2.764581                1.562856  \n",
       "std             0.753773                1.315491  \n",
       "min             0.000000                0.000000  \n",
       "25%             2.300000                1.000000  \n",
       "50%             2.780000                1.000000  \n",
       "75%             3.270000                2.000000  \n",
       "max             5.400000                9.000000  "
      ]
     },
     "execution_count": 6,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.describe()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "id": "52dd8048",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "iVBORw0KGgoAAAANSUhEUgAAAYQAAAFZCAYAAACYHVgiAAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjUuMSwgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/YYfK9AAAACXBIWXMAAAsTAAALEwEAmpwYAAAixklEQVR4nO3df3RU9Z3/8ecQKMVgSTYMAxiDB4nhRwPsYoFFpfxSjCCKWkE9yrIiLnb9kV0VAqK7aAso6GG3qFjdlvqjVaLdBYVQqFBBMeF0i/EoJ8JSjVBITCQxiQZpmO8fjvN1DAiJIZOQ5+OcnJO5930/876cHF7zuZ87M4GKioowkqQ2r128G5AktQwGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJgPbxbuBUU7duZbxbUDNLyJoW7xakJuEMQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIEGAiSpAgDQZIExDEQHn74YUaPHs2ZZ57J2WefzZQpU3j33XdjambNmkVSUlLMz7hx42JqDh06xF133UXv3r3p2bMnU6dOZd++fTE1FRUVzJw5k7S0NNLS0pg5cyYVFRUn+xQlqVWJWyBs3bqVG2+8kfXr17N69Wrat2/P5ZdfzsGDB2PqRo0aRVFRUfRn1apVMftzcnJYs2YNTz31FGvXrqWqqoopU6ZQV1cXrZkxYwaFhYWsWrWK3NxcCgsLufnmm5vlPCWptWgfryd+6aWXYh6vWLGCtLQ03nzzTbKysqLbO3bsSCgUOuoYlZWVPP300yxfvpzRo0dHx8nMzGTz5s2MHTuWoqIiNm7cSF5eHsOGDQPgkUceISsri127dpGenn6SzlCSWpcWs4ZQXV3NkSNHSEpKitm+bds2+vTpw5AhQ7jtttv46KOPovt27NjB4cOHGTNmTHRbamoqGRkZ5OfnA1BQUEDnzp2jYQAwfPhwEhMTozWSpDjOEL5uzpw5ZGZmMnTo0Oi2cePGcemll9KrVy+Ki4t54IEHmDRpEps3b6Zjx46UlpaSkJBASkpKzFjBYJDS0lIASktLSUlJIRAIRPcHAgG6du0arTmaXbt2Neo8ejfqKLVmjf1bkZrb8a6ItIhAmDt3Lm+++SZ5eXkkJCREt1955ZXR3wcMGMDgwYPJzMxk/fr1TJo06ZjjhcPhegFwvJqva+ylpLrdbzTqOLVeXnbUqSLul4xycnJ48cUXWb16NWedddY31vbo0YOePXuyZ88eALp160ZdXR3l5eUxdWVlZQSDwWhNWVkZ4XA4uj8cDlNeXh6tkSTFORBmz55Nbm4uq1ev5pxzzjlufXl5Ofv3748uMg8ePJgOHTqwadOmaM2+ffsoKiqKrhkMHTqU6upqCgoKojUFBQXU1NTErCtIUlsXt0tGd955J88//zzPPPMMSUlJlJSUAJCYmEjnzp2prq5m0aJFTJo0iVAoRHFxMQsWLCAYDDJx4kQAunTpwvXXX8+9995LMBgkOTmZefPmMWDAAEaNGgVARkYG48aNIzs7m2XLlhEOh8nOzmb8+PFO9SXpKwIVFRXh45c1va/fTfSl2bNnk5OTw2effcZ1111HYWEhlZWVhEIhLrjgAubNm0dqamq0vra2lvnz55Obm0ttbS0jR45k6dKlMTUHDx5k9uzZrFu3DoCsrCwefPDBY/bwbdStW9nkY6plS8iaFu8WpCYRt0A4VRkIbY+BoFNF3BeVJUktg4EgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElShIEgSQIMBElSRNwC4eGHH2b06NGceeaZnH322UyZMoV33303piYcDrNw4UL69u1L9+7dmTBhAjt37oypOXToEHfddRe9e/emZ8+eTJ06lX379sXUVFRUMHPmTNLS0khLS2PmzJlUVFSc7FOUpFYlboGwdetWbrzxRtavX8/q1atp3749l19+OQcPHozWLFu2jOXLl7N48WJeffVVgsEgkydPpqqqKlqTk5PDmjVreOqpp1i7di1VVVVMmTKFurq6aM2MGTMoLCxk1apV5ObmUlhYyM0339ys5ytJLV2goqIiHO8mAKqrq0lLS+PZZ58lKyuLcDhM3759uemmm7jzzjsB+Oyzz0hPT+f+++9n+vTpVFZW0qdPH5YvX87VV18NwN69e8nMzCQ3N5exY8dSVFTEsGHDyMvLY/jw4QBs27aNrKwstm/fTnp6epOeR926lU06nlq+hKxp8W5BahItZg2hurqaI0eOkJSUBMAHH3xASUkJY8aMidZ06tSJESNGkJ+fD8COHTs4fPhwTE1qaioZGRnRmoKCAjp37sywYcOiNcOHDycxMTFaI0mC9vFu4Etz5swhMzOToUOHAlBSUgJAMBiMqQsGg+zfvx+A0tJSEhISSElJqVdTWloarUlJSSEQCET3BwIBunbtGq05ml27djXqPHo36ii1Zo39W5Ga2/GuiLSIQJg7dy5vvvkmeXl5JCQkxOz76n/k8MVC89e3fd3Xa45Wf7xxGnspqW73G406Tq1XU192lOIl7peMcnJyePHFF1m9ejVnnXVWdHsoFAKo9yq+rKwsOmvo1q0bdXV1lJeXf2NNWVkZ4fD/XyoJh8OUl5fXm31IUlsW10CYPXs2ubm5rF69mnPOOSdmX69evQiFQmzatCm6rba2lm3btkXXAwYPHkyHDh1iavbt2xddSAYYOnQo1dXVFBQURGsKCgqoqamJWVeQpLYubpeM7rzzTp5//nmeeeYZkpKSomsGiYmJdO7cmUAgwKxZs1i6dCnp6en06dOHJUuWkJiYyFVXXQVAly5duP7667n33nsJBoMkJyczb948BgwYwKhRowDIyMhg3LhxZGdns2zZMsLhMNnZ2YwfP96pviR9RdxuO/3ybqKvmz17Njk5OcAXl3YWLVrEL3/5SyoqKhgyZAhLliyhf//+0fra2lrmz59Pbm4utbW1jBw5kqVLl5KamhqtOXjwILNnz2bdunUAZGVl8eCDDx6zh2/D207bHm871amixbwP4VRhILQ9BoJOFXFfVJYktQwGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJMBAkSREGgiQJaGAgDBo0iLVr1x5zf15eHoMGDfrWTUmSml+DAqG4uJiamppj7q+pqeHDDz/81k1Jkppfgy8ZBQKBY+7bvXs3p59++rdqSJIUH+2PV/Dcc8/x61//Ovp4yZIlrFy5sl5dRUUF7777LuPHj2/aDiVJzeK4gVBTU0NJSUn0cWVlJUeOHImpCQQCnHbaaUybNo05c+Y0fZeSpJMuUFFRET7R4oEDB7Jo0SIuueSSk9lTq1a3rv7sSae2hKxp8W5BahLHnSF8VWFh4cnqQ5IUZw0KhC9VVVWxd+9eDh48SDhcf4Jx3nnnfevGJEnNq0GBcPDgQWbPns1vf/tb6urq6u0Ph8MEAgE+/vjjJmtQktQ8GhQI2dnZvPzyy9x0002cd955JCUlnaS2JEnNrUGBsHHjRm6++WZ+8pOfnKx+JElx0qA3pn3nO9/h7LPPPlm9SJLiqEGBcNlll7Fhw4aT1YskKY4aFAi33norBw4c4J/+6Z/Yvn07Bw4c4KOPPqr3I0lqfRr0xrTk5GQCgUD0bqJjact3GfnGtLbHN6bpVNGgReW77777G4NAktR6NSgQcnJymvTJX3/9df7zP/+Tt956i/3797N8+XKuu+666P5Zs2bFfLAewLnnnsvGjRujjw8dOsQ999zDiy++SG1tLSNHjmTp0qWcccYZ0ZqKigruvvtu8vLyALj44ot58MEHvW1Wkr4irt+YVlNTQ//+/Vm0aBGdOnU6as2oUaMoKiqK/qxatSpmf05ODmvWrOGpp55i7dq1VFVVMWXKlJg3zs2YMYPCwkJWrVpFbm4uhYWF3HzzzSf13CSptWnQDGHx4sXHrQkEAtx9990nNN5FF13ERRddBMAtt9xy1JqOHTsSCoWOuq+yspKnn36a5cuXM3r0aABWrFhBZmYmmzdvZuzYsRQVFbFx40by8vIYNmwYAI888ghZWVns2rWL9PT0E+pVkk51DQqERYsWHXPfVxebTzQQTsS2bdvo06cPXbp04bzzzmP+/PkEg0EAduzYweHDhxkzZky0PjU1lYyMDPLz8xk7diwFBQV07tw5GgYAw4cPJzExkfz8fANBkiIa/FlGX3fkyBGKi4tZsWIF+fn55ObmNllz48aN49JLL6VXr14UFxfzwAMPMGnSJDZv3kzHjh0pLS0lISGBlJSUmOOCwSClpaUAlJaWkpKSErMYHggE6Nq1a7TmaHbt2tWonns36ii1Zo39W5Ga2/FeADfq006/ql27dpx11lksXLiQ6dOnM2fOHJ544olvOywAV155ZfT3AQMGMHjwYDIzM1m/fj2TJk065nFfvy32aHdGHe/W2cbOHOp2v9Go49R6OcvUqaJJF5UvuOAC1q9f35RDxujRowc9e/Zkz549AHTr1o26ujrKy8tj6srKyqKXlbp160ZZWVnMx3SHw2HKy8ujNZKkJg6EXbt2HfX7EZpKeXk5+/fvjy4yDx48mA4dOrBp06Zozb59+ygqKoquGQwdOpTq6moKCgqiNQUFBdTU1MSsK0hSW9egS0avv/76UbdXVlayZcsWfv7zn3P55Zef8HjV1dXRV/tHjhxh7969FBYWkpycTHJyMosWLWLSpEmEQiGKi4tZsGABwWCQiRMnAtClSxeuv/567r33XoLBIMnJycybN48BAwYwatQoADIyMhg3bhzZ2dksW7aMcDhMdnY248ePd6ovSV/RqI+u+LpwOExCQgJXXnklixcvPuE3fG3ZsoVLL7203vZrrrmGhx9+mOuuu47CwkIqKysJhUJccMEFzJs3j9TU1GhtbW0t8+fPJzc3N+aNaV+t+fKLfdatWwdAVlbWSXtjmh9d0fb40RU6VTQoELZu3Vp/gECApKQk0tLSOP3005u0udbIQGh7DASdKhp0yej8888/WX1IkuKsUbedVlVVsXXrVoqLiwFIS0vj/PPPd4YgSa1YgwNhxYoVPPDAA9TU1MTcUZSYmMj8+fP9jCBJaqUaFAi/+c1vmDNnDkOGDGHWrFlkZGQQDod57733ePzxx8nJySE5OZmrr776ZPUrSTpJGrSofMEFF5CYmMjLL79M+/axWfLXv/6ViRMnUlNTw5YtW5q80dbCReW2x0VlnSoa9Ma0Xbt2ccUVV9QLA4D27dtzxRVXsHv37iZrTpLUfBoUCImJiZSUlBxzf0lJCaeddtq3bkqS1PwaFAhjxoxhxYoVR70ktHXrVp544gnGjh3bZM1JkppPg9YQ9u7dy/jx49m/fz8DBw7knHPOAeC9996jsLCQHj168Lvf/S7m6yvbGtcQ2h7XEHSqaNAMITU1lS1btnDLLbfw6aefsnr1alavXs2nn37Kj3/8Y7Zs2dKmw0CSWrMGzRBqamr4+OOPOfPMM4+6/8MPPyQlJaVNryM4Q2h7nCHoVNGgGcLcuXO59tprj7n/uuuuY/78+d+6KUlS82tQIGzatCn60dNHM3HiRH7/+99/66YkSc2vQYFQUlJC9+7dj7k/FApx4MCBb92UJKn5NSgQunbtys6dO4+5f+fOnXTp0uVbNyVJan4NCoQLL7yQlStXkp+fX2/f9u3bWblyJRdeeGGTNSdJaj4NusuopKSEMWPGcODAAcaNG0f//v0JBAK88847bNy4kVAoxO9//3t69OhxMntu0bzLqO3xLiOdKhoUCAClpaXcd999vPLKK1RVVQFw+umnM3HiRO677z5CodBJabS1MBDaHgNBp4oGfx9Ct27deOyxxwiHw5SVlREOhwkGg0f9rmVJUuvRqG9Mgy++SzkYDDZlL5KkOGrQorIk6dRlIEiSAANBkhRhIEiSAANBkhRhIEiSAANBkhRhIEiSAANBkhRhIEiSAANBkhRhIEiSAANBkhRhIEiSAANBkhRhIEiSAANBkhRhIEiSgDgHwuuvv87UqVPp168fSUlJPPvsszH7w+EwCxcupG/fvnTv3p0JEyawc+fOmJpDhw5x11130bt3b3r27MnUqVPZt29fTE1FRQUzZ84kLS2NtLQ0Zs6cSUVFxck+PUlqVeIaCDU1NfTv359FixbRqVOnevuXLVvG8uXLWbx4Ma+++irBYJDJkydTVVUVrcnJyWHNmjU89dRTrF27lqqqKqZMmUJdXV20ZsaMGRQWFrJq1Spyc3MpLCzk5ptvbpZzlKTWIlBRURGOdxMAZ5xxBg8++CDXXXcd8MXsoG/fvtx0003ceeedAHz22Wekp6dz//33M336dCorK+nTpw/Lly/n6quvBmDv3r1kZmaSm5vL2LFjKSoqYtiwYeTl5TF8+HAAtm3bRlZWFtu3byc9Pb1Jz6Nu3comHU8tX0LWtHi3IDWJFruG8MEHH1BSUsKYMWOi2zp16sSIESPIz88HYMeOHRw+fDimJjU1lYyMjGhNQUEBnTt3ZtiwYdGa4cOHk5iYGK2RJEH7eDdwLCUlJQAEg8GY7cFgkP379wNQWlpKQkICKSkp9WpKS0ujNSkpKQQCgej+QCBA165dozVHs2vXrkb13btRR6k1a+zfitTcjndFpMUGwpe++h85fHEp6evbvu7rNUerP944jb2UVLf7jUYdp9arqS87SvHSYi8ZhUIhgHqv4svKyqKzhm7dulFXV0d5efk31pSVlREO//+lknA4THl5eb3ZhyS1ZS02EHr16kUoFGLTpk3RbbW1tWzbti26HjB48GA6dOgQU7Nv377oQjLA0KFDqa6upqCgIFpTUFBATU1NzLqCJLV1cb1kVF1dzZ49ewA4cuQIe/fupbCwkOTkZM4880xmzZrF0qVLSU9Pp0+fPixZsoTExESuuuoqALp06cL111/PvffeSzAYJDk5mXnz5jFgwABGjRoFQEZGBuPGjSM7O5tly5YRDofJzs5m/PjxTvUl6Svietvpli1buPTSS+ttv+aaa3jssccIh8MsWrSIX/7yl1RUVDBkyBCWLFlC//79o7W1tbXMnz+f3NxcamtrGTlyJEuXLiU1NTVac/DgQWbPns26desAyMrK4sEHHyQpKanJz8nbTtsebzvVqaLFvA/hVGEgtD0Ggk4VLXYNQZLUvAwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRJgIEiSIgwESRLQwgNh4cKFJCUlxfycc8450f3hcJiFCxfSt29funfvzoQJE9i5c2fMGIcOHeKuu+6id+/e9OzZk6lTp7Jv377mPhVJavFadCAApKenU1RUFP154403ovuWLVvG8uXLWbx4Ma+++irBYJDJkydTVVUVrcnJyWHNmjU89dRTrF27lqqqKqZMmUJdXV08TkeSWqwWHwjt27cnFApFf7p27Qp8MTt47LHHuOOOO7jsssvo378/jz32GNXV1eTm5gJQWVnJ008/zYIFCxg9ejSDBw9mxYoVvPPOO2zevDmOZyVJLU+LD4T333+ffv36MXDgQP7xH/+R999/H4APPviAkpISxowZE63t1KkTI0aMID8/H4AdO3Zw+PDhmJrU1FQyMjKiNZKkL7SPdwPf5Nxzz+XRRx8lPT2dsrIyHnroIS666CLefPNNSkpKAAgGgzHHBINB9u/fD0BpaSkJCQmkpKTUqyktLf3G5961a1ejeu7dqKPUmjX2b0Vqbunp6d+4v0UHwoUXXhjz+Nxzz2Xw4ME899xz/OAHPwAgEAjE1ITD4Xrbvu5Eao73D3csdbvfOH6RTimN/VtpCrt+NCluz634SV+1+qSM2+IvGX1V586d6du3L3v27CEUCgHUe6VfVlYWnTV069aNuro6ysvLj1kjSfpCqwqE2tpadu3aRSgUolevXoRCITZt2hSzf9u2bQwbNgyAwYMH06FDh5iaffv2UVRUFK2RJH2hRV8yuueee7j44otJTU2NriF8+umnXHPNNQQCAWbNmsXSpUtJT0+nT58+LFmyhMTERK666ioAunTpwvXXX8+9995LMBgkOTmZefPmMWDAAEaNGhXfk5OkFqZFB8Jf/vIXZsyYQXl5OV27duXcc89lw4YNpKWlAXD77bfz2Wefcdddd1FRUcGQIUN46aWXOP3006Nj/PSnPyUhIYHp06dTW1vLyJEjefzxx0lISIjXaUlSixSoqKgIx7uJU0ndupXxbkHNLCFrWtye20XltslFZUnSSWUgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFNGmAuHJJ59k4MCBhEIhfvjDH/LGG2/EuyVJajHaTCC89NJLzJkzh3/913/ltddeY+jQofzoRz/iww8/jHdrktQitJlAWL58Oddeey3Tpk0jIyODhx56iFAoxH/913/FuzVJahHax7uB5vD555+zY8cObr311pjtY8aMIT8/v0mfKyFrWpOOJ32T9FWr492CTiFtYoZQXl5OXV0dwWAwZnswGKS0tDROXUlSy9ImAuFLgUAg5nE4HK63TZLaqjYRCCkpKSQkJNSbDZSVldWbNUhSW9UmAuE73/kOgwcPZtOmTTHbN23axLBhw+LUlSS1LG0iEAB+/OMf89xzz/GrX/2KoqIiZs+ezYEDB5g+fXq8W2vVtmzZQlJSEuXl5fFuRdK31CbuMgK44oor+Pjjj3nooYcoKSmhX79+vPDCC6SlpcW7tRZh1qxZ/PrXv663/bXXXmPgwIFx6EhtQVJS0jfuv+aaa3jssceapxm1nUAAmDFjBjNmzIh3Gy3WqFGjWLFiRcy2lJSUOHWjtqCoqCj6+/r167nttttitn33u9+NqT98+DAdOnRotv7amjZzyUjH17FjR0KhUMzP448/zogRI+jZsyf9+vXj1ltvpaKi4phjVFZWMnPmTPr06UMoFGLQoEE8+uijMftvv/12+vTpQ2pqKpdccgl/+tOfmuHs1BJ99W+tS5cuMdtqa2vp1asXubm5XHrppXTv3p1f/OIXPPvss5xxxhkx4xzt0mV+fj6XXHIJPXr0oF+/fvzLv/wLn3zySbOeX2tjIOgbtWvXjoULF7Jt2zZ+/vOf88c//pG77777mPUPPPAA7777Ls8//zwFBQX87Gc/o2fPnsAXt/lOmTKF/fv38/zzz/Paa68xYsQIJk2axIEDB5rrlNTK/Pu//zszZszgzTffZMKECSd0zDvvvMMVV1xBVlYWW7du5emnn+btt9/mn//5n09yt61bm7pkpG+2cePGmFdef//3f09ubm70ca9evViwYAHXXnstjz/+OO3a1X898eGHHzJw4ECGDBkSPeZLr732Gm+//Ta7d++mU6dOANxzzz3k5eXx/PPPc/vtt5+sU1MrNnPmTC677LIGHfMf//EfTJ48OebTCZYuXcrIkSP56KOPvN38GAwERY0YMYJly5ZFH3/3u9/lD3/4A4888gjvvfcen3zyCXV1dXz++eeUlJTQo0ePemPceOONTJs2jbfeeovRo0dz8cUXc/755wPw1ltv8emnn9KnT5+YY2pra/nzn/98ck9Ordbf/u3fNviYt956iz179vDb3/42ui0cDgPw5z//2UA4BgNBUaeddhq9e/eOPi4uLmbKlCnccMMNzJ07l7/5m7/hrbfe4sYbb+Tzzz8/6hgXXnghb7/9Nhs2bOAPf/gDU6ZM4bLLLuPRRx/lyJEjdOvWjXXr1tU77vTTTz9p56XWLTExMeZxu3btov+5f+mvf/1rzOMjR45www03cMstt9Qb72gvZPQFA0HH9Kc//YnPP/+chQsXkpCQAEBeXt5xj0tJSWHq1KlMnTqVCy+8kBtvvJFHHnmEQYMGUVpaSrt27TjrrLNOcvc6VXXt2pVPP/2UTz75hO9973sAvP322zE1gwYNYufOnTEvcHR8LirrmM4++2yOHDnCo48+yvvvv09ubi6PP/74Nx7zk5/8hJdffpn/+7//o6ioiDVr1nDWWWfRsWNHRo0axfDhw7n22mvZsGED77//PgUFBfz0pz/1y4p0ws4991wSExNZsGABe/bs4X/+53948sknY2puv/12/vd//5fs7Ozo5aO8vDzuuOOO+DTdShgIOqbvf//7LFq0iEcffZThw4fzq1/9ivvvv/8bj+nYsSMPPPAA559/PuPHj6e6uprf/OY3wBcfLvjCCy9wwQUXcPvtt/ODH/yA6dOns3v3bqfxOmHJyck88cQTbNq0iREjRrBy5UrmzZsXU/P973+ftWvXUlxczMSJEzn//PNZsGCBawfHEaioqAgfv0ySdKpzhiBJAgwESVKEgSBJAgwESVKEgSBJAgwESVKEgSA1kWeffZakpCS2b98e71akRjEQJEmAgSBJijAQpFbmy48gl5qagSA1wIEDB7jjjjvo378/3bp1IzMzk9tuu42qqqpozeHDh1mwYAEZGRl0796dyZMn8/7778eMk5mZyaxZs+qNP2vWLDIzM6OPP/jgA5KSknjkkUd48skn+bu/+zu6detGfn5+dM3ijTfeOO7zSSfCj7+WTlBJSQljx46lrKyMG264gf79+3PgwAFefvllPv7442jd3Llz6dSpE9nZ2ZSXl/Ozn/2MmTNn8rvf/a7Rz/3CCy9QXV3NP/zDP9C5c2e6d+9OcXHxSXs+tU0GgnSC/u3f/o2//OUvvPLKK4wYMSK6PScnJ+YLW0477TRefvnl6FeMJicnM3fuXHbu3Em/fv0a9dzFxcX88Y9/pHv37tFtBQUFJ+351DZ5yUg6AUeOHOGVV15h3LhxMWHwpUAgEP19+vTpMd83fd555wF8q8s4EyZMiAmDrzoZz6e2yUCQTkBZWRmffPIJ/fv3P27tmWeeGfM4KSkJgIMHDzb6+b/pG+ZOxvOpbTIQpBPw5SWhr84EjuXLrxs91hjfNE5dXd1Rt3fq1OlbPZ90IgwE6QQEg0G+973v8e677zbJeElJSVRWVtbb/uGHHzbJ+FJjGAjSCWjXrh0TJkxgw4YN5Ofn19vf0FfjvXv3Zvv27Rw6dCi6bceOHUcdW2ou3mUknaD77ruPzZs3c9lllzFt2jT69etHaWkpa9as4ZlnnmnQWNOnT+e///u/mTx5MldeeSX79+/nF7/4BX379o15T4PUnJwhSCeoe/fubNy4kSuuuIKXXnqJu+++m2eeeYYhQ4aQkpLSoLF++MMfsnjxYoqLi5k7dy4bNmzgySefZNCgQSepe+n4AhUVFa48SZKcIUiSvmAgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIAA0GSFGEgSJIA+H+Pub/M+MTtwQAAAABJRU5ErkJggg==\n",
      "text/plain": [
       "<Figure size 360x360 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# apply the Fivethirtyeight style to plots.\n",
    "plt.style.use(\"fivethirtyeight\")\n",
    "\n",
    "# display a frequency distribution for churn.\n",
    "plt.figure(figsize=(5, 5))\n",
    "ax = sns.countplot(x=df['churn'], palette='Reds', linewidth=1)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "dfa46db3",
   "metadata": {},
   "source": [
    "The plot shows a class imbalance of the data between churners and non-churners. To address this, resampling would be a suitable approach. To keep this case simple, the imbalance is kept forward and specific metrics are chosen for model evaluations."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "id": "f8f7d64b",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "False    0.855086\n",
       "True     0.144914\n",
       "Name: churn, dtype: float64"
      ]
     },
     "execution_count": 8,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df['churn'].value_counts(normalize=True)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "1fdfad72",
   "metadata": {},
   "source": [
    "It looks like they're losing 14.4% of their customers and retaining aroung 85.5% annually. Let's see what this looks like by each state."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "id": "d0ebd061",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>state</th>\n",
       "      <th>churn</th>\n",
       "      <th>percent</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>AK</td>\n",
       "      <td>False</td>\n",
       "      <td>0.942308</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>AK</td>\n",
       "      <td>True</td>\n",
       "      <td>0.057692</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>AL</td>\n",
       "      <td>False</td>\n",
       "      <td>0.900000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>AL</td>\n",
       "      <td>True</td>\n",
       "      <td>0.100000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>AR</td>\n",
       "      <td>False</td>\n",
       "      <td>0.800000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>97</th>\n",
       "      <td>WI</td>\n",
       "      <td>True</td>\n",
       "      <td>0.089744</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>98</th>\n",
       "      <td>WV</td>\n",
       "      <td>False</td>\n",
       "      <td>0.905660</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>99</th>\n",
       "      <td>WV</td>\n",
       "      <td>True</td>\n",
       "      <td>0.094340</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>100</th>\n",
       "      <td>WY</td>\n",
       "      <td>False</td>\n",
       "      <td>0.883117</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>101</th>\n",
       "      <td>WY</td>\n",
       "      <td>True</td>\n",
       "      <td>0.116883</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>102 rows ?? 3 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "    state  churn   percent\n",
       "0      AK  False  0.942308\n",
       "1      AK   True  0.057692\n",
       "2      AL  False  0.900000\n",
       "3      AL   True  0.100000\n",
       "4      AR  False  0.800000\n",
       "..    ...    ...       ...\n",
       "97     WI   True  0.089744\n",
       "98     WV  False  0.905660\n",
       "99     WV   True  0.094340\n",
       "100    WY  False  0.883117\n",
       "101    WY   True  0.116883\n",
       "\n",
       "[102 rows x 3 columns]"
      ]
     },
     "execution_count": 9,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "states = df.groupby('state')['churn'].value_counts(normalize=True)\n",
    "states = pd.DataFrame(states)\n",
    "states.columns = ['percent']\n",
    "states = states.reset_index()\n",
    "states"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "id": "1523d029",
   "metadata": {},
   "outputs": [],
   "source": [
    "# create a function to name a column and add columns as necessary\n",
    "def combine_col(name, *cols):\n",
    "    df[name] = sum(cols)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "id": "6e4c89de",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>state</th>\n",
       "      <th>churn</th>\n",
       "      <th>percent</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>NJ</td>\n",
       "      <td>True</td>\n",
       "      <td>0.264706</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>CA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.264706</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>TX</td>\n",
       "      <td>True</td>\n",
       "      <td>0.250000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>MD</td>\n",
       "      <td>True</td>\n",
       "      <td>0.242857</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>SC</td>\n",
       "      <td>True</td>\n",
       "      <td>0.233333</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>MI</td>\n",
       "      <td>True</td>\n",
       "      <td>0.219178</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>MS</td>\n",
       "      <td>True</td>\n",
       "      <td>0.215385</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>NV</td>\n",
       "      <td>True</td>\n",
       "      <td>0.212121</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>WA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.212121</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>9</th>\n",
       "      <td>ME</td>\n",
       "      <td>True</td>\n",
       "      <td>0.209677</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>MT</td>\n",
       "      <td>True</td>\n",
       "      <td>0.205882</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>11</th>\n",
       "      <td>AR</td>\n",
       "      <td>True</td>\n",
       "      <td>0.200000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>12</th>\n",
       "      <td>KS</td>\n",
       "      <td>True</td>\n",
       "      <td>0.185714</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>13</th>\n",
       "      <td>NY</td>\n",
       "      <td>True</td>\n",
       "      <td>0.180723</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>14</th>\n",
       "      <td>MN</td>\n",
       "      <td>True</td>\n",
       "      <td>0.178571</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>15</th>\n",
       "      <td>PA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.177778</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>16</th>\n",
       "      <td>MA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.169231</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>17</th>\n",
       "      <td>CT</td>\n",
       "      <td>True</td>\n",
       "      <td>0.162162</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>18</th>\n",
       "      <td>NC</td>\n",
       "      <td>True</td>\n",
       "      <td>0.161765</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>19</th>\n",
       "      <td>NH</td>\n",
       "      <td>True</td>\n",
       "      <td>0.160714</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>20</th>\n",
       "      <td>GA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.148148</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>21</th>\n",
       "      <td>DE</td>\n",
       "      <td>True</td>\n",
       "      <td>0.147541</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>22</th>\n",
       "      <td>OK</td>\n",
       "      <td>True</td>\n",
       "      <td>0.147541</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>23</th>\n",
       "      <td>OR</td>\n",
       "      <td>True</td>\n",
       "      <td>0.141026</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>24</th>\n",
       "      <td>UT</td>\n",
       "      <td>True</td>\n",
       "      <td>0.138889</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>25</th>\n",
       "      <td>CO</td>\n",
       "      <td>True</td>\n",
       "      <td>0.136364</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>26</th>\n",
       "      <td>KY</td>\n",
       "      <td>True</td>\n",
       "      <td>0.135593</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>27</th>\n",
       "      <td>SD</td>\n",
       "      <td>True</td>\n",
       "      <td>0.133333</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>28</th>\n",
       "      <td>OH</td>\n",
       "      <td>True</td>\n",
       "      <td>0.128205</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>29</th>\n",
       "      <td>FL</td>\n",
       "      <td>True</td>\n",
       "      <td>0.126984</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>30</th>\n",
       "      <td>IN</td>\n",
       "      <td>True</td>\n",
       "      <td>0.126761</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>31</th>\n",
       "      <td>ID</td>\n",
       "      <td>True</td>\n",
       "      <td>0.123288</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>32</th>\n",
       "      <td>WY</td>\n",
       "      <td>True</td>\n",
       "      <td>0.116883</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>33</th>\n",
       "      <td>MO</td>\n",
       "      <td>True</td>\n",
       "      <td>0.111111</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>34</th>\n",
       "      <td>VT</td>\n",
       "      <td>True</td>\n",
       "      <td>0.109589</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>35</th>\n",
       "      <td>AL</td>\n",
       "      <td>True</td>\n",
       "      <td>0.100000</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>36</th>\n",
       "      <td>NM</td>\n",
       "      <td>True</td>\n",
       "      <td>0.096774</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>37</th>\n",
       "      <td>ND</td>\n",
       "      <td>True</td>\n",
       "      <td>0.096774</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>38</th>\n",
       "      <td>WV</td>\n",
       "      <td>True</td>\n",
       "      <td>0.094340</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>39</th>\n",
       "      <td>TN</td>\n",
       "      <td>True</td>\n",
       "      <td>0.094340</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>40</th>\n",
       "      <td>DC</td>\n",
       "      <td>True</td>\n",
       "      <td>0.092593</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>41</th>\n",
       "      <td>RI</td>\n",
       "      <td>True</td>\n",
       "      <td>0.092308</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>42</th>\n",
       "      <td>WI</td>\n",
       "      <td>True</td>\n",
       "      <td>0.089744</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>43</th>\n",
       "      <td>IL</td>\n",
       "      <td>True</td>\n",
       "      <td>0.086207</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>44</th>\n",
       "      <td>NE</td>\n",
       "      <td>True</td>\n",
       "      <td>0.081967</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>45</th>\n",
       "      <td>LA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.078431</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>46</th>\n",
       "      <td>IA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.068182</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>47</th>\n",
       "      <td>VA</td>\n",
       "      <td>True</td>\n",
       "      <td>0.064935</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>48</th>\n",
       "      <td>AZ</td>\n",
       "      <td>True</td>\n",
       "      <td>0.062500</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>49</th>\n",
       "      <td>AK</td>\n",
       "      <td>True</td>\n",
       "      <td>0.057692</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>50</th>\n",
       "      <td>HI</td>\n",
       "      <td>True</td>\n",
       "      <td>0.056604</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   state  churn   percent\n",
       "0     NJ   True  0.264706\n",
       "1     CA   True  0.264706\n",
       "2     TX   True  0.250000\n",
       "3     MD   True  0.242857\n",
       "4     SC   True  0.233333\n",
       "5     MI   True  0.219178\n",
       "6     MS   True  0.215385\n",
       "7     NV   True  0.212121\n",
       "8     WA   True  0.212121\n",
       "9     ME   True  0.209677\n",
       "10    MT   True  0.205882\n",
       "11    AR   True  0.200000\n",
       "12    KS   True  0.185714\n",
       "13    NY   True  0.180723\n",
       "14    MN   True  0.178571\n",
       "15    PA   True  0.177778\n",
       "16    MA   True  0.169231\n",
       "17    CT   True  0.162162\n",
       "18    NC   True  0.161765\n",
       "19    NH   True  0.160714\n",
       "20    GA   True  0.148148\n",
       "21    DE   True  0.147541\n",
       "22    OK   True  0.147541\n",
       "23    OR   True  0.141026\n",
       "24    UT   True  0.138889\n",
       "25    CO   True  0.136364\n",
       "26    KY   True  0.135593\n",
       "27    SD   True  0.133333\n",
       "28    OH   True  0.128205\n",
       "29    FL   True  0.126984\n",
       "30    IN   True  0.126761\n",
       "31    ID   True  0.123288\n",
       "32    WY   True  0.116883\n",
       "33    MO   True  0.111111\n",
       "34    VT   True  0.109589\n",
       "35    AL   True  0.100000\n",
       "36    NM   True  0.096774\n",
       "37    ND   True  0.096774\n",
       "38    WV   True  0.094340\n",
       "39    TN   True  0.094340\n",
       "40    DC   True  0.092593\n",
       "41    RI   True  0.092308\n",
       "42    WI   True  0.089744\n",
       "43    IL   True  0.086207\n",
       "44    NE   True  0.081967\n",
       "45    LA   True  0.078431\n",
       "46    IA   True  0.068182\n",
       "47    VA   True  0.064935\n",
       "48    AZ   True  0.062500\n",
       "49    AK   True  0.057692\n",
       "50    HI   True  0.056604"
      ]
     },
     "execution_count": 11,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Finding the churn rates for each state\n",
    "churn_rate_state = states.loc[states['churn'] == True].sort_values(\"percent\", ascending =False)\\\n",
    "                                                        .reset_index().drop(\"index\", axis =1)\n",
    "churn_rate_state"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "26e8a214",
   "metadata": {},
   "source": [
    "There are definitely states with a higher churn rate. some are losing more than a fifth of their customers each year. Let's create a category for those states."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "id": "d887b8d9",
   "metadata": {},
   "outputs": [],
   "source": [
    "colors = plotly.colors.DEFAULT_PLOTLY_COLORS\n",
    "churn_dict = {0: \"no churn\", 1: \"churn\"}"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 13,
   "id": "0f4aee4d",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "application/vnd.plotly.v1+json": {
       "config": {
        "linkText": "Export to plot.ly",
        "plotlyServerURL": "https://plot.ly",
        "showLink": false
       },
       "data": [
        {
         "marker": {
          "color": "rgb(31, 119, 180)"
         },
         "name": "no churn",
         "type": "bar",
         "x": [
          "AK",
          "AL",
          "AR",
          "AZ",
          "CA",
          "CO",
          "CT",
          "DC",
          "DE",
          "FL",
          "GA",
          "HI",
          "IA",
          "ID",
          "IL",
          "IN",
          "KS",
          "KY",
          "LA",
          "MA",
          "MD",
          "ME",
          "MI",
          "MN",
          "MO",
          "MS",
          "MT",
          "NC",
          "ND",
          "NE",
          "NH",
          "NJ",
          "NM",
          "NV",
          "NY",
          "OH",
          "OK",
          "OR",
          "PA",
          "RI",
          "SC",
          "SD",
          "TN",
          "TX",
          "UT",
          "VA",
          "VT",
          "WA",
          "WI",
          "WV",
          "WY"
         ],
         "y": [
          49,
          72,
          44,
          60,
          25,
          57,
          62,
          49,
          52,
          55,
          46,
          50,
          41,
          64,
          53,
          62,
          57,
          51,
          47,
          54,
          53,
          49,
          57,
          69,
          56,
          51,
          54,
          57,
          56,
          56,
          47,
          50,
          56,
          52,
          68,
          68,
          52,
          67,
          37,
          59,
          46,
          52,
          48,
          54,
          62,
          72,
          65,
          52,
          71,
          96,
          68
         ]
        },
        {
         "marker": {
          "color": "rgb(255, 127, 14)"
         },
         "name": "churn",
         "type": "bar",
         "x": [
          "AK",
          "AL",
          "AR",
          "AZ",
          "CA",
          "CO",
          "CT",
          "DC",
          "DE",
          "FL",
          "GA",
          "HI",
          "IA",
          "ID",
          "IL",
          "IN",
          "KS",
          "KY",
          "LA",
          "MA",
          "MD",
          "ME",
          "MI",
          "MN",
          "MO",
          "MS",
          "MT",
          "NC",
          "ND",
          "NE",
          "NH",
          "NJ",
          "NM",
          "NV",
          "NY",
          "OH",
          "OK",
          "OR",
          "PA",
          "RI",
          "SC",
          "SD",
          "TN",
          "TX",
          "UT",
          "VA",
          "VT",
          "WA",
          "WI",
          "WV",
          "WY"
         ],
         "y": [
          3,
          8,
          11,
          4,
          9,
          9,
          12,
          5,
          9,
          8,
          8,
          3,
          3,
          9,
          5,
          9,
          13,
          8,
          4,
          11,
          17,
          13,
          16,
          15,
          7,
          14,
          14,
          11,
          6,
          5,
          9,
          18,
          6,
          14,
          15,
          10,
          9,
          11,
          8,
          6,
          14,
          8,
          5,
          18,
          10,
          5,
          8,
          14,
          7,
          10,
          9
         ]
        }
       ],
       "layout": {
        "autosize": true,
        "barmode": "stack",
        "legend": {
         "x": 0,
         "y": 1
        },
        "margin": {
         "l": 50,
         "r": 50
        },
        "template": {
         "data": {
          "bar": [
           {
            "error_x": {
             "color": "#2a3f5f"
            },
            "error_y": {
             "color": "#2a3f5f"
            },
            "marker": {
             "line": {
              "color": "#E5ECF6",
              "width": 0.5
             },
             "pattern": {
              "fillmode": "overlay",
              "size": 10,
              "solidity": 0.2
             }
            },
            "type": "bar"
           }
          ],
          "barpolar": [
           {
            "marker": {
             "line": {
              "color": "#E5ECF6",
              "width": 0.5
             },
             "pattern": {
              "fillmode": "overlay",
              "size": 10,
              "solidity": 0.2
             }
            },
            "type": "barpolar"
           }
          ],
          "carpet": [
           {
            "aaxis": {
             "endlinecolor": "#2a3f5f",
             "gridcolor": "white",
             "linecolor": "white",
             "minorgridcolor": "white",
             "startlinecolor": "#2a3f5f"
            },
            "baxis": {
             "endlinecolor": "#2a3f5f",
             "gridcolor": "white",
             "linecolor": "white",
             "minorgridcolor": "white",
             "startlinecolor": "#2a3f5f"
            },
            "type": "carpet"
           }
          ],
          "choropleth": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "type": "choropleth"
           }
          ],
          "contour": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "contour"
           }
          ],
          "contourcarpet": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "type": "contourcarpet"
           }
          ],
          "heatmap": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "heatmap"
           }
          ],
          "heatmapgl": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "heatmapgl"
           }
          ],
          "histogram": [
           {
            "marker": {
             "pattern": {
              "fillmode": "overlay",
              "size": 10,
              "solidity": 0.2
             }
            },
            "type": "histogram"
           }
          ],
          "histogram2d": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "histogram2d"
           }
          ],
          "histogram2dcontour": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "histogram2dcontour"
           }
          ],
          "mesh3d": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "type": "mesh3d"
           }
          ],
          "parcoords": [
           {
            "line": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "parcoords"
           }
          ],
          "pie": [
           {
            "automargin": true,
            "type": "pie"
           }
          ],
          "scatter": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatter"
           }
          ],
          "scatter3d": [
           {
            "line": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatter3d"
           }
          ],
          "scattercarpet": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattercarpet"
           }
          ],
          "scattergeo": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattergeo"
           }
          ],
          "scattergl": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattergl"
           }
          ],
          "scattermapbox": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scattermapbox"
           }
          ],
          "scatterpolar": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatterpolar"
           }
          ],
          "scatterpolargl": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatterpolargl"
           }
          ],
          "scatterternary": [
           {
            "marker": {
             "colorbar": {
              "outlinewidth": 0,
              "ticks": ""
             }
            },
            "type": "scatterternary"
           }
          ],
          "surface": [
           {
            "colorbar": {
             "outlinewidth": 0,
             "ticks": ""
            },
            "colorscale": [
             [
              0,
              "#0d0887"
             ],
             [
              0.1111111111111111,
              "#46039f"
             ],
             [
              0.2222222222222222,
              "#7201a8"
             ],
             [
              0.3333333333333333,
              "#9c179e"
             ],
             [
              0.4444444444444444,
              "#bd3786"
             ],
             [
              0.5555555555555556,
              "#d8576b"
             ],
             [
              0.6666666666666666,
              "#ed7953"
             ],
             [
              0.7777777777777778,
              "#fb9f3a"
             ],
             [
              0.8888888888888888,
              "#fdca26"
             ],
             [
              1,
              "#f0f921"
             ]
            ],
            "type": "surface"
           }
          ],
          "table": [
           {
            "cells": {
             "fill": {
              "color": "#EBF0F8"
             },
             "line": {
              "color": "white"
             }
            },
            "header": {
             "fill": {
              "color": "#C8D4E3"
             },
             "line": {
              "color": "white"
             }
            },
            "type": "table"
           }
          ]
         },
         "layout": {
          "annotationdefaults": {
           "arrowcolor": "#2a3f5f",
           "arrowhead": 0,
           "arrowwidth": 1
          },
          "autotypenumbers": "strict",
          "coloraxis": {
           "colorbar": {
            "outlinewidth": 0,
            "ticks": ""
           }
          },
          "colorscale": {
           "diverging": [
            [
             0,
             "#8e0152"
            ],
            [
             0.1,
             "#c51b7d"
            ],
            [
             0.2,
             "#de77ae"
            ],
            [
             0.3,
             "#f1b6da"
            ],
            [
             0.4,
             "#fde0ef"
            ],
            [
             0.5,
             "#f7f7f7"
            ],
            [
             0.6,
             "#e6f5d0"
            ],
            [
             0.7,
             "#b8e186"
            ],
            [
             0.8,
             "#7fbc41"
            ],
            [
             0.9,
             "#4d9221"
            ],
            [
             1,
             "#276419"
            ]
           ],
           "sequential": [
            [
             0,
             "#0d0887"
            ],
            [
             0.1111111111111111,
             "#46039f"
            ],
            [
             0.2222222222222222,
             "#7201a8"
            ],
            [
             0.3333333333333333,
             "#9c179e"
            ],
            [
             0.4444444444444444,
             "#bd3786"
            ],
            [
             0.5555555555555556,
             "#d8576b"
            ],
            [
             0.6666666666666666,
             "#ed7953"
            ],
            [
             0.7777777777777778,
             "#fb9f3a"
            ],
            [
             0.8888888888888888,
             "#fdca26"
            ],
            [
             1,
             "#f0f921"
            ]
           ],
           "sequentialminus": [
            [
             0,
             "#0d0887"
            ],
            [
             0.1111111111111111,
             "#46039f"
            ],
            [
             0.2222222222222222,
             "#7201a8"
            ],
            [
             0.3333333333333333,
             "#9c179e"
            ],
            [
             0.4444444444444444,
             "#bd3786"
            ],
            [
             0.5555555555555556,
             "#d8576b"
            ],
            [
             0.6666666666666666,
             "#ed7953"
            ],
            [
             0.7777777777777778,
             "#fb9f3a"
            ],
            [
             0.8888888888888888,
             "#fdca26"
            ],
            [
             1,
             "#f0f921"
            ]
           ]
          },
          "colorway": [
           "#636efa",
           "#EF553B",
           "#00cc96",
           "#ab63fa",
           "#FFA15A",
           "#19d3f3",
           "#FF6692",
           "#B6E880",
           "#FF97FF",
           "#FECB52"
          ],
          "font": {
           "color": "#2a3f5f"
          },
          "geo": {
           "bgcolor": "white",
           "lakecolor": "white",
           "landcolor": "#E5ECF6",
           "showlakes": true,
           "showland": true,
           "subunitcolor": "white"
          },
          "hoverlabel": {
           "align": "left"
          },
          "hovermode": "closest",
          "mapbox": {
           "style": "light"
          },
          "paper_bgcolor": "white",
          "plot_bgcolor": "#E5ECF6",
          "polar": {
           "angularaxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           },
           "bgcolor": "#E5ECF6",
           "radialaxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           }
          },
          "scene": {
           "xaxis": {
            "backgroundcolor": "#E5ECF6",
            "gridcolor": "white",
            "gridwidth": 2,
            "linecolor": "white",
            "showbackground": true,
            "ticks": "",
            "zerolinecolor": "white"
           },
           "yaxis": {
            "backgroundcolor": "#E5ECF6",
            "gridcolor": "white",
            "gridwidth": 2,
            "linecolor": "white",
            "showbackground": true,
            "ticks": "",
            "zerolinecolor": "white"
           },
           "zaxis": {
            "backgroundcolor": "#E5ECF6",
            "gridcolor": "white",
            "gridwidth": 2,
            "linecolor": "white",
            "showbackground": true,
            "ticks": "",
            "zerolinecolor": "white"
           }
          },
          "shapedefaults": {
           "line": {
            "color": "#2a3f5f"
           }
          },
          "ternary": {
           "aaxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           },
           "baxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           },
           "bgcolor": "#E5ECF6",
           "caxis": {
            "gridcolor": "white",
            "linecolor": "white",
            "ticks": ""
           }
          },
          "title": {
           "x": 0.05
          },
          "xaxis": {
           "automargin": true,
           "gridcolor": "white",
           "linecolor": "white",
           "ticks": "",
           "title": {
            "standoff": 15
           },
           "zerolinecolor": "white",
           "zerolinewidth": 2
          },
          "yaxis": {
           "automargin": true,
           "gridcolor": "white",
           "linecolor": "white",
           "ticks": "",
           "title": {
            "standoff": 15
           },
           "zerolinecolor": "white",
           "zerolinewidth": 2
          }
         }
        },
        "title": {
         "text": "Churn distribution per state"
        },
        "xaxis": {
         "tickangle": 45,
         "title": {
          "text": "state"
         }
        },
        "yaxis": {
         "automargin": true,
         "title": {
          "text": "#samples"
         }
        }
       }
      },
      "text/html": [
       "<div>                            <div id=\"259c860c-0ba7-4f74-87ad-ce3b285a062d\" class=\"plotly-graph-div\" style=\"height:525px; width:100%;\"></div>            <script type=\"text/javascript\">                require([\"plotly\"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById(\"259c860c-0ba7-4f74-87ad-ce3b285a062d\")) {                    Plotly.newPlot(                        \"259c860c-0ba7-4f74-87ad-ce3b285a062d\",                        [{\"marker\":{\"color\":\"rgb(31, 119, 180)\"},\"name\":\"no churn\",\"x\":[\"AK\",\"AL\",\"AR\",\"AZ\",\"CA\",\"CO\",\"CT\",\"DC\",\"DE\",\"FL\",\"GA\",\"HI\",\"IA\",\"ID\",\"IL\",\"IN\",\"KS\",\"KY\",\"LA\",\"MA\",\"MD\",\"ME\",\"MI\",\"MN\",\"MO\",\"MS\",\"MT\",\"NC\",\"ND\",\"NE\",\"NH\",\"NJ\",\"NM\",\"NV\",\"NY\",\"OH\",\"OK\",\"OR\",\"PA\",\"RI\",\"SC\",\"SD\",\"TN\",\"TX\",\"UT\",\"VA\",\"VT\",\"WA\",\"WI\",\"WV\",\"WY\"],\"y\":[49,72,44,60,25,57,62,49,52,55,46,50,41,64,53,62,57,51,47,54,53,49,57,69,56,51,54,57,56,56,47,50,56,52,68,68,52,67,37,59,46,52,48,54,62,72,65,52,71,96,68],\"type\":\"bar\"},{\"marker\":{\"color\":\"rgb(255, 127, 14)\"},\"name\":\"churn\",\"x\":[\"AK\",\"AL\",\"AR\",\"AZ\",\"CA\",\"CO\",\"CT\",\"DC\",\"DE\",\"FL\",\"GA\",\"HI\",\"IA\",\"ID\",\"IL\",\"IN\",\"KS\",\"KY\",\"LA\",\"MA\",\"MD\",\"ME\",\"MI\",\"MN\",\"MO\",\"MS\",\"MT\",\"NC\",\"ND\",\"NE\",\"NH\",\"NJ\",\"NM\",\"NV\",\"NY\",\"OH\",\"OK\",\"OR\",\"PA\",\"RI\",\"SC\",\"SD\",\"TN\",\"TX\",\"UT\",\"VA\",\"VT\",\"WA\",\"WI\",\"WV\",\"WY\"],\"y\":[3,8,11,4,9,9,12,5,9,8,8,3,3,9,5,9,13,8,4,11,17,13,16,15,7,14,14,11,6,5,9,18,6,14,15,10,9,11,8,6,14,8,5,18,10,5,8,14,7,10,9],\"type\":\"bar\"}],                        {\"autosize\":true,\"barmode\":\"stack\",\"legend\":{\"x\":0,\"y\":1},\"margin\":{\"l\":50,\"r\":50},\"template\":{\"data\":{\"barpolar\":[{\"marker\":{\"line\":{\"color\":\"#E5ECF6\",\"width\":0.5},\"pattern\":{\"fillmode\":\"overlay\",\"size\":10,\"solidity\":0.2}},\"type\":\"barpolar\"}],\"bar\":[{\"error_x\":{\"color\":\"#2a3f5f\"},\"error_y\":{\"color\":\"#2a3f5f\"},\"marker\":{\"line\":{\"color\":\"#E5ECF6\",\"width\":0.5},\"pattern\":{\"fillmode\":\"overlay\",\"size\":10,\"solidity\":0.2}},\"type\":\"bar\"}],\"carpet\":[{\"aaxis\":{\"endlinecolor\":\"#2a3f5f\",\"gridcolor\":\"white\",\"linecolor\":\"white\",\"minorgridcolor\":\"white\",\"startlinecolor\":\"#2a3f5f\"},\"baxis\":{\"endlinecolor\":\"#2a3f5f\",\"gridcolor\":\"white\",\"linecolor\":\"white\",\"minorgridcolor\":\"white\",\"startlinecolor\":\"#2a3f5f\"},\"type\":\"carpet\"}],\"choropleth\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"type\":\"choropleth\"}],\"contourcarpet\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"type\":\"contourcarpet\"}],\"contour\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"colorscale\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]],\"type\":\"contour\"}],\"heatmapgl\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"colorscale\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]],\"type\":\"heatmapgl\"}],\"heatmap\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"colorscale\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]],\"type\":\"heatmap\"}],\"histogram2dcontour\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"colorscale\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]],\"type\":\"histogram2dcontour\"}],\"histogram2d\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"colorscale\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]],\"type\":\"histogram2d\"}],\"histogram\":[{\"marker\":{\"pattern\":{\"fillmode\":\"overlay\",\"size\":10,\"solidity\":0.2}},\"type\":\"histogram\"}],\"mesh3d\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"type\":\"mesh3d\"}],\"parcoords\":[{\"line\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"parcoords\"}],\"pie\":[{\"automargin\":true,\"type\":\"pie\"}],\"scatter3d\":[{\"line\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scatter3d\"}],\"scattercarpet\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scattercarpet\"}],\"scattergeo\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scattergeo\"}],\"scattergl\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scattergl\"}],\"scattermapbox\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scattermapbox\"}],\"scatterpolargl\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scatterpolargl\"}],\"scatterpolar\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scatterpolar\"}],\"scatter\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scatter\"}],\"scatterternary\":[{\"marker\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"type\":\"scatterternary\"}],\"surface\":[{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"},\"colorscale\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]],\"type\":\"surface\"}],\"table\":[{\"cells\":{\"fill\":{\"color\":\"#EBF0F8\"},\"line\":{\"color\":\"white\"}},\"header\":{\"fill\":{\"color\":\"#C8D4E3\"},\"line\":{\"color\":\"white\"}},\"type\":\"table\"}]},\"layout\":{\"annotationdefaults\":{\"arrowcolor\":\"#2a3f5f\",\"arrowhead\":0,\"arrowwidth\":1},\"autotypenumbers\":\"strict\",\"coloraxis\":{\"colorbar\":{\"outlinewidth\":0,\"ticks\":\"\"}},\"colorscale\":{\"diverging\":[[0,\"#8e0152\"],[0.1,\"#c51b7d\"],[0.2,\"#de77ae\"],[0.3,\"#f1b6da\"],[0.4,\"#fde0ef\"],[0.5,\"#f7f7f7\"],[0.6,\"#e6f5d0\"],[0.7,\"#b8e186\"],[0.8,\"#7fbc41\"],[0.9,\"#4d9221\"],[1,\"#276419\"]],\"sequential\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]],\"sequentialminus\":[[0.0,\"#0d0887\"],[0.1111111111111111,\"#46039f\"],[0.2222222222222222,\"#7201a8\"],[0.3333333333333333,\"#9c179e\"],[0.4444444444444444,\"#bd3786\"],[0.5555555555555556,\"#d8576b\"],[0.6666666666666666,\"#ed7953\"],[0.7777777777777778,\"#fb9f3a\"],[0.8888888888888888,\"#fdca26\"],[1.0,\"#f0f921\"]]},\"colorway\":[\"#636efa\",\"#EF553B\",\"#00cc96\",\"#ab63fa\",\"#FFA15A\",\"#19d3f3\",\"#FF6692\",\"#B6E880\",\"#FF97FF\",\"#FECB52\"],\"font\":{\"color\":\"#2a3f5f\"},\"geo\":{\"bgcolor\":\"white\",\"lakecolor\":\"white\",\"landcolor\":\"#E5ECF6\",\"showlakes\":true,\"showland\":true,\"subunitcolor\":\"white\"},\"hoverlabel\":{\"align\":\"left\"},\"hovermode\":\"closest\",\"mapbox\":{\"style\":\"light\"},\"paper_bgcolor\":\"white\",\"plot_bgcolor\":\"#E5ECF6\",\"polar\":{\"angularaxis\":{\"gridcolor\":\"white\",\"linecolor\":\"white\",\"ticks\":\"\"},\"bgcolor\":\"#E5ECF6\",\"radialaxis\":{\"gridcolor\":\"white\",\"linecolor\":\"white\",\"ticks\":\"\"}},\"scene\":{\"xaxis\":{\"backgroundcolor\":\"#E5ECF6\",\"gridcolor\":\"white\",\"gridwidth\":2,\"linecolor\":\"white\",\"showbackground\":true,\"ticks\":\"\",\"zerolinecolor\":\"white\"},\"yaxis\":{\"backgroundcolor\":\"#E5ECF6\",\"gridcolor\":\"white\",\"gridwidth\":2,\"linecolor\":\"white\",\"showbackground\":true,\"ticks\":\"\",\"zerolinecolor\":\"white\"},\"zaxis\":{\"backgroundcolor\":\"#E5ECF6\",\"gridcolor\":\"white\",\"gridwidth\":2,\"linecolor\":\"white\",\"showbackground\":true,\"ticks\":\"\",\"zerolinecolor\":\"white\"}},\"shapedefaults\":{\"line\":{\"color\":\"#2a3f5f\"}},\"ternary\":{\"aaxis\":{\"gridcolor\":\"white\",\"linecolor\":\"white\",\"ticks\":\"\"},\"baxis\":{\"gridcolor\":\"white\",\"linecolor\":\"white\",\"ticks\":\"\"},\"bgcolor\":\"#E5ECF6\",\"caxis\":{\"gridcolor\":\"white\",\"linecolor\":\"white\",\"ticks\":\"\"}},\"title\":{\"x\":0.05},\"xaxis\":{\"automargin\":true,\"gridcolor\":\"white\",\"linecolor\":\"white\",\"ticks\":\"\",\"title\":{\"standoff\":15},\"zerolinecolor\":\"white\",\"zerolinewidth\":2},\"yaxis\":{\"automargin\":true,\"gridcolor\":\"white\",\"linecolor\":\"white\",\"ticks\":\"\",\"title\":{\"standoff\":15},\"zerolinecolor\":\"white\",\"zerolinewidth\":2}}},\"title\":{\"text\":\"Churn distribution per state\"},\"xaxis\":{\"tickangle\":45,\"title\":{\"text\":\"state\"}},\"yaxis\":{\"automargin\":true,\"title\":{\"text\":\"#samples\"}}},                        {\"responsive\": true}                    ).then(function(){\n",
       "                            \n",
       "var gd = document.getElementById('259c860c-0ba7-4f74-87ad-ce3b285a062d');\n",
       "var x = new MutationObserver(function (mutations, observer) {{\n",
       "        var display = window.getComputedStyle(gd).display;\n",
       "        if (!display || display === 'none') {{\n",
       "            console.log([gd, 'removed!']);\n",
       "            Plotly.purge(gd);\n",
       "            observer.disconnect();\n",
       "        }}\n",
       "}});\n",
       "\n",
       "// Listen for the removal of the full notebook cells\n",
       "var notebookContainer = gd.closest('#notebook-container');\n",
       "if (notebookContainer) {{\n",
       "    x.observe(notebookContainer, {childList: true});\n",
       "}}\n",
       "\n",
       "// Listen for the clearing of the current output cell\n",
       "var outputEl = gd.closest('.output');\n",
       "if (outputEl) {{\n",
       "    x.observe(outputEl, {childList: true});\n",
       "}}\n",
       "\n",
       "                        })                };                });            </script>        </div>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "state_churn_df = df.groupby([\"state\", \"churn\"]).size().unstack()\n",
    "trace1 = go.Bar(\n",
    "    x=state_churn_df.index,\n",
    "    y=state_churn_df[0],\n",
    "    marker = dict(color = colors[0]),\n",
    "    name='no churn'\n",
    ")\n",
    "trace2 = go.Bar(\n",
    "    x=state_churn_df.index,\n",
    "    y=state_churn_df[1],\n",
    "    marker = dict(color = colors[1]),\n",
    "    name='churn'\n",
    ")\n",
    "data = [trace1, trace2]\n",
    "layout = go.Layout(\n",
    "    title='Churn distribution per state',\n",
    "    autosize=True,\n",
    "    barmode='stack',\n",
    "    margin=go.layout.Margin(l=50, r=50),\n",
    "    xaxis=dict(\n",
    "        title='state',\n",
    "        tickangle=45\n",
    "    ),\n",
    "    yaxis=dict(\n",
    "        title='#samples',\n",
    "        automargin=True,\n",
    "    ),\n",
    "    legend=dict(\n",
    "        x=0,\n",
    "        y=1,\n",
    "    ),\n",
    ")\n",
    "fig = go.Figure(data=data, layout=layout)\n",
    "iplot(fig, filename='stacked-bar')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "67ad2452",
   "metadata": {},
   "source": [
    "We can see some states have a higher proportion of churn than others such as NJ, CA, TX, SC, MD, SC, MI, MS, NV, WA, ME, MT, AR. Others have a low churn rate. We should incorporate the state into our analysis to help predict if the customer is going to churn."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "id": "d9c32105",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "['NJ', 'CA', 'TX', 'MD', 'SC', 'MI', 'MS', 'NV', 'WA', 'ME', 'MT', 'AR']"
      ]
     },
     "execution_count": 14,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "high_churn = churn_rate_state.loc[churn_rate_state['percent'] >= .2]\n",
    "state_high_churn = list(high_churn['state'])\n",
    "state_high_churn"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "id": "094ce567",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "['KS', 'NY', 'MN', 'PA', 'MA', 'CT', 'NC', 'NH']"
      ]
     },
     "execution_count": 15,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# category for mid churn rates\n",
    "mid_churn = churn_rate_state.loc[(churn_rate_state['percent'] < .2)&(churn_rate_state['percent'] >=.15)]\n",
    "mid_churn_states = list(mid_churn['state'])\n",
    "mid_churn_states"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 16,
   "id": "f2451049",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "['KS', 'NY', 'MN', 'PA', 'MA', 'CT', 'NC', 'NH']"
      ]
     },
     "execution_count": 16,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# category for mid churn rates\n",
    "mid_churn = churn_rate_state.loc[(churn_rate_state['percent'] < .2)&(churn_rate_state['percent'] >=.15)]\n",
    "mid_churn_states = list(mid_churn['state'])\n",
    "mid_churn_states"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "id": "622eb03f",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "['GA',\n",
       " 'DE',\n",
       " 'OK',\n",
       " 'OR',\n",
       " 'UT',\n",
       " 'CO',\n",
       " 'KY',\n",
       " 'SD',\n",
       " 'OH',\n",
       " 'FL',\n",
       " 'IN',\n",
       " 'ID',\n",
       " 'WY',\n",
       " 'MO',\n",
       " 'VT',\n",
       " 'AL']"
      ]
     },
     "execution_count": 17,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# lower churn rate states category\n",
    "mid_low_churn = churn_rate_state.loc[(churn_rate_state['percent'] <.15)&(churn_rate_state['percent'] >= .1)]\n",
    "mid_low_churn_states = list(mid_low_churn['state'])\n",
    "mid_low_churn_states"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "id": "accbed54",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "['NM',\n",
       " 'ND',\n",
       " 'WV',\n",
       " 'TN',\n",
       " 'DC',\n",
       " 'RI',\n",
       " 'WI',\n",
       " 'IL',\n",
       " 'NE',\n",
       " 'LA',\n",
       " 'IA',\n",
       " 'VA',\n",
       " 'AZ',\n",
       " 'AK',\n",
       " 'HI']"
      ]
     },
     "execution_count": 18,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "low_churn = churn_rate_state.loc[churn_rate_state['percent'] <.1]\n",
    "low_churn_states = list(low_churn['state'])\n",
    "low_churn_states"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 19,
   "id": "c71b04da",
   "metadata": {},
   "outputs": [],
   "source": [
    "# assign states to high, medium, medium-low and low churn rate categories\n",
    "\n",
    "def categorize(state):\n",
    "    if state in state_high_churn:\n",
    "        state = 'high'\n",
    "    elif state in mid_churn_states:\n",
    "        state='med'\n",
    "    elif state in mid_low_churn_states:\n",
    "        state = 'med-low'\n",
    "    else:\n",
    "         state = 'low'\n",
    "    return state"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "id": "f2cb9552",
   "metadata": {},
   "outputs": [],
   "source": [
    "def competition(df):\n",
    "\n",
    "    df[\"state_churn_rate\"] = df['state'].apply(categorize)\n",
    "    return df"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 21,
   "id": "4dd12a60",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>state</th>\n",
       "      <th>account length</th>\n",
       "      <th>area code</th>\n",
       "      <th>phone number</th>\n",
       "      <th>international plan</th>\n",
       "      <th>voice mail plan</th>\n",
       "      <th>number vmail messages</th>\n",
       "      <th>total day minutes</th>\n",
       "      <th>total day calls</th>\n",
       "      <th>total day charge</th>\n",
       "      <th>total eve minutes</th>\n",
       "      <th>total eve calls</th>\n",
       "      <th>total eve charge</th>\n",
       "      <th>total night minutes</th>\n",
       "      <th>total night calls</th>\n",
       "      <th>total night charge</th>\n",
       "      <th>total intl minutes</th>\n",
       "      <th>total intl calls</th>\n",
       "      <th>total intl charge</th>\n",
       "      <th>customer service calls</th>\n",
       "      <th>churn</th>\n",
       "      <th>state_churn_rate</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>KS</td>\n",
       "      <td>128</td>\n",
       "      <td>415</td>\n",
       "      <td>382-4657</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>25</td>\n",
       "      <td>265.1</td>\n",
       "      <td>110</td>\n",
       "      <td>45.07</td>\n",
       "      <td>197.4</td>\n",
       "      <td>99</td>\n",
       "      <td>16.78</td>\n",
       "      <td>244.7</td>\n",
       "      <td>91</td>\n",
       "      <td>11.01</td>\n",
       "      <td>10.0</td>\n",
       "      <td>3</td>\n",
       "      <td>2.70</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>med</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>OH</td>\n",
       "      <td>107</td>\n",
       "      <td>415</td>\n",
       "      <td>371-7191</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>26</td>\n",
       "      <td>161.6</td>\n",
       "      <td>123</td>\n",
       "      <td>27.47</td>\n",
       "      <td>195.5</td>\n",
       "      <td>103</td>\n",
       "      <td>16.62</td>\n",
       "      <td>254.4</td>\n",
       "      <td>103</td>\n",
       "      <td>11.45</td>\n",
       "      <td>13.7</td>\n",
       "      <td>3</td>\n",
       "      <td>3.70</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>NJ</td>\n",
       "      <td>137</td>\n",
       "      <td>415</td>\n",
       "      <td>358-1921</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>243.4</td>\n",
       "      <td>114</td>\n",
       "      <td>41.38</td>\n",
       "      <td>121.2</td>\n",
       "      <td>110</td>\n",
       "      <td>10.30</td>\n",
       "      <td>162.6</td>\n",
       "      <td>104</td>\n",
       "      <td>7.32</td>\n",
       "      <td>12.2</td>\n",
       "      <td>5</td>\n",
       "      <td>3.29</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "      <td>high</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>OH</td>\n",
       "      <td>84</td>\n",
       "      <td>408</td>\n",
       "      <td>375-9999</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>299.4</td>\n",
       "      <td>71</td>\n",
       "      <td>50.90</td>\n",
       "      <td>61.9</td>\n",
       "      <td>88</td>\n",
       "      <td>5.26</td>\n",
       "      <td>196.9</td>\n",
       "      <td>89</td>\n",
       "      <td>8.86</td>\n",
       "      <td>6.6</td>\n",
       "      <td>7</td>\n",
       "      <td>1.78</td>\n",
       "      <td>2</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>OK</td>\n",
       "      <td>75</td>\n",
       "      <td>415</td>\n",
       "      <td>330-6626</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>166.7</td>\n",
       "      <td>113</td>\n",
       "      <td>28.34</td>\n",
       "      <td>148.3</td>\n",
       "      <td>122</td>\n",
       "      <td>12.61</td>\n",
       "      <td>186.9</td>\n",
       "      <td>121</td>\n",
       "      <td>8.41</td>\n",
       "      <td>10.1</td>\n",
       "      <td>3</td>\n",
       "      <td>2.73</td>\n",
       "      <td>3</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3328</th>\n",
       "      <td>AZ</td>\n",
       "      <td>192</td>\n",
       "      <td>415</td>\n",
       "      <td>414-4276</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>36</td>\n",
       "      <td>156.2</td>\n",
       "      <td>77</td>\n",
       "      <td>26.55</td>\n",
       "      <td>215.5</td>\n",
       "      <td>126</td>\n",
       "      <td>18.32</td>\n",
       "      <td>279.1</td>\n",
       "      <td>83</td>\n",
       "      <td>12.56</td>\n",
       "      <td>9.9</td>\n",
       "      <td>6</td>\n",
       "      <td>2.67</td>\n",
       "      <td>2</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3329</th>\n",
       "      <td>WV</td>\n",
       "      <td>68</td>\n",
       "      <td>415</td>\n",
       "      <td>370-3271</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>231.1</td>\n",
       "      <td>57</td>\n",
       "      <td>39.29</td>\n",
       "      <td>153.4</td>\n",
       "      <td>55</td>\n",
       "      <td>13.04</td>\n",
       "      <td>191.3</td>\n",
       "      <td>123</td>\n",
       "      <td>8.61</td>\n",
       "      <td>9.6</td>\n",
       "      <td>4</td>\n",
       "      <td>2.59</td>\n",
       "      <td>3</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3330</th>\n",
       "      <td>RI</td>\n",
       "      <td>28</td>\n",
       "      <td>510</td>\n",
       "      <td>328-8230</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>180.8</td>\n",
       "      <td>109</td>\n",
       "      <td>30.74</td>\n",
       "      <td>288.8</td>\n",
       "      <td>58</td>\n",
       "      <td>24.55</td>\n",
       "      <td>191.9</td>\n",
       "      <td>91</td>\n",
       "      <td>8.64</td>\n",
       "      <td>14.1</td>\n",
       "      <td>6</td>\n",
       "      <td>3.81</td>\n",
       "      <td>2</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3331</th>\n",
       "      <td>CT</td>\n",
       "      <td>184</td>\n",
       "      <td>510</td>\n",
       "      <td>364-6381</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>213.8</td>\n",
       "      <td>105</td>\n",
       "      <td>36.35</td>\n",
       "      <td>159.6</td>\n",
       "      <td>84</td>\n",
       "      <td>13.57</td>\n",
       "      <td>139.2</td>\n",
       "      <td>137</td>\n",
       "      <td>6.26</td>\n",
       "      <td>5.0</td>\n",
       "      <td>10</td>\n",
       "      <td>1.35</td>\n",
       "      <td>2</td>\n",
       "      <td>False</td>\n",
       "      <td>med</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3332</th>\n",
       "      <td>TN</td>\n",
       "      <td>74</td>\n",
       "      <td>415</td>\n",
       "      <td>400-4344</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>25</td>\n",
       "      <td>234.4</td>\n",
       "      <td>113</td>\n",
       "      <td>39.85</td>\n",
       "      <td>265.9</td>\n",
       "      <td>82</td>\n",
       "      <td>22.60</td>\n",
       "      <td>241.4</td>\n",
       "      <td>77</td>\n",
       "      <td>10.86</td>\n",
       "      <td>13.7</td>\n",
       "      <td>4</td>\n",
       "      <td>3.70</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>3333 rows ?? 22 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "     state  account length  area code phone number international plan  \\\n",
       "0       KS             128        415     382-4657                 no   \n",
       "1       OH             107        415     371-7191                 no   \n",
       "2       NJ             137        415     358-1921                 no   \n",
       "3       OH              84        408     375-9999                yes   \n",
       "4       OK              75        415     330-6626                yes   \n",
       "...    ...             ...        ...          ...                ...   \n",
       "3328    AZ             192        415     414-4276                 no   \n",
       "3329    WV              68        415     370-3271                 no   \n",
       "3330    RI              28        510     328-8230                 no   \n",
       "3331    CT             184        510     364-6381                yes   \n",
       "3332    TN              74        415     400-4344                 no   \n",
       "\n",
       "     voice mail plan  number vmail messages  total day minutes  \\\n",
       "0                yes                     25              265.1   \n",
       "1                yes                     26              161.6   \n",
       "2                 no                      0              243.4   \n",
       "3                 no                      0              299.4   \n",
       "4                 no                      0              166.7   \n",
       "...              ...                    ...                ...   \n",
       "3328             yes                     36              156.2   \n",
       "3329              no                      0              231.1   \n",
       "3330              no                      0              180.8   \n",
       "3331              no                      0              213.8   \n",
       "3332             yes                     25              234.4   \n",
       "\n",
       "      total day calls  total day charge  total eve minutes  total eve calls  \\\n",
       "0                 110             45.07              197.4               99   \n",
       "1                 123             27.47              195.5              103   \n",
       "2                 114             41.38              121.2              110   \n",
       "3                  71             50.90               61.9               88   \n",
       "4                 113             28.34              148.3              122   \n",
       "...               ...               ...                ...              ...   \n",
       "3328               77             26.55              215.5              126   \n",
       "3329               57             39.29              153.4               55   \n",
       "3330              109             30.74              288.8               58   \n",
       "3331              105             36.35              159.6               84   \n",
       "3332              113             39.85              265.9               82   \n",
       "\n",
       "      total eve charge  total night minutes  total night calls  \\\n",
       "0                16.78                244.7                 91   \n",
       "1                16.62                254.4                103   \n",
       "2                10.30                162.6                104   \n",
       "3                 5.26                196.9                 89   \n",
       "4                12.61                186.9                121   \n",
       "...                ...                  ...                ...   \n",
       "3328             18.32                279.1                 83   \n",
       "3329             13.04                191.3                123   \n",
       "3330             24.55                191.9                 91   \n",
       "3331             13.57                139.2                137   \n",
       "3332             22.60                241.4                 77   \n",
       "\n",
       "      total night charge  total intl minutes  total intl calls  \\\n",
       "0                  11.01                10.0                 3   \n",
       "1                  11.45                13.7                 3   \n",
       "2                   7.32                12.2                 5   \n",
       "3                   8.86                 6.6                 7   \n",
       "4                   8.41                10.1                 3   \n",
       "...                  ...                 ...               ...   \n",
       "3328               12.56                 9.9                 6   \n",
       "3329                8.61                 9.6                 4   \n",
       "3330                8.64                14.1                 6   \n",
       "3331                6.26                 5.0                10   \n",
       "3332               10.86                13.7                 4   \n",
       "\n",
       "      total intl charge  customer service calls  churn state_churn_rate  \n",
       "0                  2.70                       1  False              med  \n",
       "1                  3.70                       1  False          med-low  \n",
       "2                  3.29                       0  False             high  \n",
       "3                  1.78                       2  False          med-low  \n",
       "4                  2.73                       3  False          med-low  \n",
       "...                 ...                     ...    ...              ...  \n",
       "3328               2.67                       2  False              low  \n",
       "3329               2.59                       3  False              low  \n",
       "3330               3.81                       2  False              low  \n",
       "3331               1.35                       2  False              med  \n",
       "3332               3.70                       0  False              low  \n",
       "\n",
       "[3333 rows x 22 columns]"
      ]
     },
     "execution_count": 21,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "competition(df)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 22,
   "id": "805b2ffe",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Finding the total domestic minutes for each customer\n",
    "combine_col(\"total_dom_minutes\",\n",
    "           df['total day minutes'],\n",
    "           df['total eve minutes'],\n",
    "           df['total night minutes'])"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 23,
   "id": "1449116b",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Finding the total domestic charges for each customer\n",
    "combine_col('total_dom_charges',\n",
    "           df['total day charge'],\n",
    "           df['total eve charge'],\n",
    "           df['total night charge'])"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "fe556d56",
   "metadata": {},
   "source": [
    "# 3. Data Cleaning"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 24,
   "id": "571ac7a1",
   "metadata": {
    "scrolled": false
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "state                     0\n",
       "account length            0\n",
       "area code                 0\n",
       "phone number              0\n",
       "international plan        0\n",
       "voice mail plan           0\n",
       "number vmail messages     0\n",
       "total day minutes         0\n",
       "total day calls           0\n",
       "total day charge          0\n",
       "total eve minutes         0\n",
       "total eve calls           0\n",
       "total eve charge          0\n",
       "total night minutes       0\n",
       "total night calls         0\n",
       "total night charge        0\n",
       "total intl minutes        0\n",
       "total intl calls          0\n",
       "total intl charge         0\n",
       "customer service calls    0\n",
       "churn                     0\n",
       "state_churn_rate          0\n",
       "total_dom_minutes         0\n",
       "total_dom_charges         0\n",
       "dtype: int64"
      ]
     },
     "execution_count": 24,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "#check for missing values\n",
    "df.isnull().sum()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "28f7a1a3",
   "metadata": {},
   "source": [
    "#### No Missing values observed"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 25,
   "id": "97bd26d1",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0"
      ]
     },
     "execution_count": 25,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# check for duplicated rows\n",
    "df.duplicated().sum()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "091d978b",
   "metadata": {},
   "source": [
    "#### No dulicated rows observed"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 26,
   "id": "cd318302",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "account length            0.096606\n",
       "area code                 1.126823\n",
       "number vmail messages     1.264824\n",
       "total day minutes        -0.029077\n",
       "total day calls          -0.111787\n",
       "total day charge         -0.029083\n",
       "total eve minutes        -0.023877\n",
       "total eve calls          -0.055563\n",
       "total eve charge         -0.023858\n",
       "total night minutes       0.008921\n",
       "total night calls         0.032500\n",
       "total night charge        0.008886\n",
       "total intl minutes       -0.245136\n",
       "total intl calls          1.321478\n",
       "total intl charge        -0.245287\n",
       "customer service calls    1.091359\n",
       "churn                     2.018356\n",
       "total_dom_minutes        -0.036998\n",
       "total_dom_charges        -0.033641\n",
       "dtype: float64"
      ]
     },
     "execution_count": 26,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# check for outliers\n",
    "df.skew()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "edf6e00b",
   "metadata": {},
   "source": [
    "* churn has a high positive correlation with 'customer service calls' and 'area code' has a positive correlation with 'total intl calls'. "
   ]
  },
  {
   "cell_type": "markdown",
   "id": "1334e6e9",
   "metadata": {},
   "source": [
    "* churn has a high positive correlation with 'customer service calls' and 'area code' has a positive correlation with 'total intl calls'. "
   ]
  },
  {
   "cell_type": "markdown",
   "id": "35269505",
   "metadata": {},
   "source": [
    "# 4. Feature Engineering\n",
    "## Feature Extraction"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 27,
   "id": "0f934ac2",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "count    3333.000000\n",
       "mean        8.099010\n",
       "std        13.688365\n",
       "min         0.000000\n",
       "25%         0.000000\n",
       "50%         0.000000\n",
       "75%        20.000000\n",
       "max        51.000000\n",
       "Name: number vmail messages, dtype: float64"
      ]
     },
     "execution_count": 27,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df['number vmail messages'].describe()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 28,
   "id": "6a16bddd",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "count    922.000000\n",
       "mean      29.277657\n",
       "std        7.559027\n",
       "min        4.000000\n",
       "25%       24.000000\n",
       "50%       29.000000\n",
       "75%       34.000000\n",
       "max       51.000000\n",
       "Name: number vmail messages, dtype: float64"
      ]
     },
     "execution_count": 28,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df['number vmail messages'][df['number vmail messages']>0].describe()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 29,
   "id": "1a9bf5c1",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>state</th>\n",
       "      <th>account length</th>\n",
       "      <th>area code</th>\n",
       "      <th>phone number</th>\n",
       "      <th>international plan</th>\n",
       "      <th>voice mail plan</th>\n",
       "      <th>number vmail messages</th>\n",
       "      <th>total day minutes</th>\n",
       "      <th>total day calls</th>\n",
       "      <th>total day charge</th>\n",
       "      <th>total eve minutes</th>\n",
       "      <th>total eve calls</th>\n",
       "      <th>total eve charge</th>\n",
       "      <th>total night minutes</th>\n",
       "      <th>total night calls</th>\n",
       "      <th>total night charge</th>\n",
       "      <th>total intl minutes</th>\n",
       "      <th>total intl calls</th>\n",
       "      <th>total intl charge</th>\n",
       "      <th>customer service calls</th>\n",
       "      <th>churn</th>\n",
       "      <th>state_churn_rate</th>\n",
       "      <th>total_dom_minutes</th>\n",
       "      <th>total_dom_charges</th>\n",
       "      <th>vmail_messages</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>KS</td>\n",
       "      <td>128</td>\n",
       "      <td>415</td>\n",
       "      <td>382-4657</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>25</td>\n",
       "      <td>265.1</td>\n",
       "      <td>110</td>\n",
       "      <td>45.07</td>\n",
       "      <td>197.4</td>\n",
       "      <td>99</td>\n",
       "      <td>16.78</td>\n",
       "      <td>244.7</td>\n",
       "      <td>91</td>\n",
       "      <td>11.01</td>\n",
       "      <td>10.0</td>\n",
       "      <td>3</td>\n",
       "      <td>2.70</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>med</td>\n",
       "      <td>707.2</td>\n",
       "      <td>72.86</td>\n",
       "      <td>Normal Users</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>OH</td>\n",
       "      <td>107</td>\n",
       "      <td>415</td>\n",
       "      <td>371-7191</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>26</td>\n",
       "      <td>161.6</td>\n",
       "      <td>123</td>\n",
       "      <td>27.47</td>\n",
       "      <td>195.5</td>\n",
       "      <td>103</td>\n",
       "      <td>16.62</td>\n",
       "      <td>254.4</td>\n",
       "      <td>103</td>\n",
       "      <td>11.45</td>\n",
       "      <td>13.7</td>\n",
       "      <td>3</td>\n",
       "      <td>3.70</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "      <td>611.5</td>\n",
       "      <td>55.54</td>\n",
       "      <td>Normal Users</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>NJ</td>\n",
       "      <td>137</td>\n",
       "      <td>415</td>\n",
       "      <td>358-1921</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>243.4</td>\n",
       "      <td>114</td>\n",
       "      <td>41.38</td>\n",
       "      <td>121.2</td>\n",
       "      <td>110</td>\n",
       "      <td>10.30</td>\n",
       "      <td>162.6</td>\n",
       "      <td>104</td>\n",
       "      <td>7.32</td>\n",
       "      <td>12.2</td>\n",
       "      <td>5</td>\n",
       "      <td>3.29</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "      <td>high</td>\n",
       "      <td>527.2</td>\n",
       "      <td>59.00</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>OH</td>\n",
       "      <td>84</td>\n",
       "      <td>408</td>\n",
       "      <td>375-9999</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>299.4</td>\n",
       "      <td>71</td>\n",
       "      <td>50.90</td>\n",
       "      <td>61.9</td>\n",
       "      <td>88</td>\n",
       "      <td>5.26</td>\n",
       "      <td>196.9</td>\n",
       "      <td>89</td>\n",
       "      <td>8.86</td>\n",
       "      <td>6.6</td>\n",
       "      <td>7</td>\n",
       "      <td>1.78</td>\n",
       "      <td>2</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "      <td>558.2</td>\n",
       "      <td>65.02</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>OK</td>\n",
       "      <td>75</td>\n",
       "      <td>415</td>\n",
       "      <td>330-6626</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>166.7</td>\n",
       "      <td>113</td>\n",
       "      <td>28.34</td>\n",
       "      <td>148.3</td>\n",
       "      <td>122</td>\n",
       "      <td>12.61</td>\n",
       "      <td>186.9</td>\n",
       "      <td>121</td>\n",
       "      <td>8.41</td>\n",
       "      <td>10.1</td>\n",
       "      <td>3</td>\n",
       "      <td>2.73</td>\n",
       "      <td>3</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "      <td>501.9</td>\n",
       "      <td>49.36</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>AL</td>\n",
       "      <td>118</td>\n",
       "      <td>510</td>\n",
       "      <td>391-8027</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>223.4</td>\n",
       "      <td>98</td>\n",
       "      <td>37.98</td>\n",
       "      <td>220.6</td>\n",
       "      <td>101</td>\n",
       "      <td>18.75</td>\n",
       "      <td>203.9</td>\n",
       "      <td>118</td>\n",
       "      <td>9.18</td>\n",
       "      <td>6.3</td>\n",
       "      <td>6</td>\n",
       "      <td>1.70</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "      <td>647.9</td>\n",
       "      <td>65.91</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>MA</td>\n",
       "      <td>121</td>\n",
       "      <td>510</td>\n",
       "      <td>355-9993</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>24</td>\n",
       "      <td>218.2</td>\n",
       "      <td>88</td>\n",
       "      <td>37.09</td>\n",
       "      <td>348.5</td>\n",
       "      <td>108</td>\n",
       "      <td>29.62</td>\n",
       "      <td>212.6</td>\n",
       "      <td>118</td>\n",
       "      <td>9.57</td>\n",
       "      <td>7.5</td>\n",
       "      <td>7</td>\n",
       "      <td>2.03</td>\n",
       "      <td>3</td>\n",
       "      <td>False</td>\n",
       "      <td>med</td>\n",
       "      <td>779.3</td>\n",
       "      <td>76.28</td>\n",
       "      <td>Normal Users</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>MO</td>\n",
       "      <td>147</td>\n",
       "      <td>415</td>\n",
       "      <td>329-9001</td>\n",
       "      <td>yes</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>157.0</td>\n",
       "      <td>79</td>\n",
       "      <td>26.69</td>\n",
       "      <td>103.1</td>\n",
       "      <td>94</td>\n",
       "      <td>8.76</td>\n",
       "      <td>211.8</td>\n",
       "      <td>96</td>\n",
       "      <td>9.53</td>\n",
       "      <td>7.1</td>\n",
       "      <td>6</td>\n",
       "      <td>1.92</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "      <td>471.9</td>\n",
       "      <td>44.98</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>LA</td>\n",
       "      <td>117</td>\n",
       "      <td>408</td>\n",
       "      <td>335-4719</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>184.5</td>\n",
       "      <td>97</td>\n",
       "      <td>31.37</td>\n",
       "      <td>351.6</td>\n",
       "      <td>80</td>\n",
       "      <td>29.89</td>\n",
       "      <td>215.8</td>\n",
       "      <td>90</td>\n",
       "      <td>9.71</td>\n",
       "      <td>8.7</td>\n",
       "      <td>4</td>\n",
       "      <td>2.35</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "      <td>751.9</td>\n",
       "      <td>70.97</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>9</th>\n",
       "      <td>WV</td>\n",
       "      <td>141</td>\n",
       "      <td>415</td>\n",
       "      <td>330-8173</td>\n",
       "      <td>yes</td>\n",
       "      <td>yes</td>\n",
       "      <td>37</td>\n",
       "      <td>258.6</td>\n",
       "      <td>84</td>\n",
       "      <td>43.96</td>\n",
       "      <td>222.0</td>\n",
       "      <td>111</td>\n",
       "      <td>18.87</td>\n",
       "      <td>326.4</td>\n",
       "      <td>97</td>\n",
       "      <td>14.69</td>\n",
       "      <td>11.2</td>\n",
       "      <td>5</td>\n",
       "      <td>3.02</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "      <td>807.0</td>\n",
       "      <td>77.52</td>\n",
       "      <td>Normal Users</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>IN</td>\n",
       "      <td>65</td>\n",
       "      <td>415</td>\n",
       "      <td>329-6603</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>129.1</td>\n",
       "      <td>137</td>\n",
       "      <td>21.95</td>\n",
       "      <td>228.5</td>\n",
       "      <td>83</td>\n",
       "      <td>19.42</td>\n",
       "      <td>208.8</td>\n",
       "      <td>111</td>\n",
       "      <td>9.40</td>\n",
       "      <td>12.7</td>\n",
       "      <td>6</td>\n",
       "      <td>3.43</td>\n",
       "      <td>4</td>\n",
       "      <td>True</td>\n",
       "      <td>med-low</td>\n",
       "      <td>566.4</td>\n",
       "      <td>50.77</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>11</th>\n",
       "      <td>RI</td>\n",
       "      <td>74</td>\n",
       "      <td>415</td>\n",
       "      <td>344-9403</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>187.7</td>\n",
       "      <td>127</td>\n",
       "      <td>31.91</td>\n",
       "      <td>163.4</td>\n",
       "      <td>148</td>\n",
       "      <td>13.89</td>\n",
       "      <td>196.0</td>\n",
       "      <td>94</td>\n",
       "      <td>8.82</td>\n",
       "      <td>9.1</td>\n",
       "      <td>5</td>\n",
       "      <td>2.46</td>\n",
       "      <td>0</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "      <td>547.1</td>\n",
       "      <td>54.62</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>12</th>\n",
       "      <td>IA</td>\n",
       "      <td>168</td>\n",
       "      <td>408</td>\n",
       "      <td>363-1107</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>128.8</td>\n",
       "      <td>96</td>\n",
       "      <td>21.90</td>\n",
       "      <td>104.9</td>\n",
       "      <td>71</td>\n",
       "      <td>8.92</td>\n",
       "      <td>141.1</td>\n",
       "      <td>128</td>\n",
       "      <td>6.35</td>\n",
       "      <td>11.2</td>\n",
       "      <td>2</td>\n",
       "      <td>3.02</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "      <td>374.8</td>\n",
       "      <td>37.17</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>13</th>\n",
       "      <td>MT</td>\n",
       "      <td>95</td>\n",
       "      <td>510</td>\n",
       "      <td>394-8006</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>156.6</td>\n",
       "      <td>88</td>\n",
       "      <td>26.62</td>\n",
       "      <td>247.6</td>\n",
       "      <td>75</td>\n",
       "      <td>21.05</td>\n",
       "      <td>192.3</td>\n",
       "      <td>115</td>\n",
       "      <td>8.65</td>\n",
       "      <td>12.3</td>\n",
       "      <td>5</td>\n",
       "      <td>3.32</td>\n",
       "      <td>3</td>\n",
       "      <td>False</td>\n",
       "      <td>high</td>\n",
       "      <td>596.5</td>\n",
       "      <td>56.32</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>14</th>\n",
       "      <td>IA</td>\n",
       "      <td>62</td>\n",
       "      <td>415</td>\n",
       "      <td>366-9238</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>120.7</td>\n",
       "      <td>70</td>\n",
       "      <td>20.52</td>\n",
       "      <td>307.2</td>\n",
       "      <td>76</td>\n",
       "      <td>26.11</td>\n",
       "      <td>203.0</td>\n",
       "      <td>99</td>\n",
       "      <td>9.14</td>\n",
       "      <td>13.1</td>\n",
       "      <td>6</td>\n",
       "      <td>3.54</td>\n",
       "      <td>4</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "      <td>630.9</td>\n",
       "      <td>55.77</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>15</th>\n",
       "      <td>NY</td>\n",
       "      <td>161</td>\n",
       "      <td>415</td>\n",
       "      <td>351-7269</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>332.9</td>\n",
       "      <td>67</td>\n",
       "      <td>56.59</td>\n",
       "      <td>317.8</td>\n",
       "      <td>97</td>\n",
       "      <td>27.01</td>\n",
       "      <td>160.6</td>\n",
       "      <td>128</td>\n",
       "      <td>7.23</td>\n",
       "      <td>5.4</td>\n",
       "      <td>9</td>\n",
       "      <td>1.46</td>\n",
       "      <td>4</td>\n",
       "      <td>True</td>\n",
       "      <td>med</td>\n",
       "      <td>811.3</td>\n",
       "      <td>90.83</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>16</th>\n",
       "      <td>ID</td>\n",
       "      <td>85</td>\n",
       "      <td>408</td>\n",
       "      <td>350-8884</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>27</td>\n",
       "      <td>196.4</td>\n",
       "      <td>139</td>\n",
       "      <td>33.39</td>\n",
       "      <td>280.9</td>\n",
       "      <td>90</td>\n",
       "      <td>23.88</td>\n",
       "      <td>89.3</td>\n",
       "      <td>75</td>\n",
       "      <td>4.02</td>\n",
       "      <td>13.8</td>\n",
       "      <td>4</td>\n",
       "      <td>3.73</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "      <td>566.6</td>\n",
       "      <td>61.29</td>\n",
       "      <td>Normal Users</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>17</th>\n",
       "      <td>VT</td>\n",
       "      <td>93</td>\n",
       "      <td>510</td>\n",
       "      <td>386-2923</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>190.7</td>\n",
       "      <td>114</td>\n",
       "      <td>32.42</td>\n",
       "      <td>218.2</td>\n",
       "      <td>111</td>\n",
       "      <td>18.55</td>\n",
       "      <td>129.6</td>\n",
       "      <td>121</td>\n",
       "      <td>5.83</td>\n",
       "      <td>8.1</td>\n",
       "      <td>3</td>\n",
       "      <td>2.19</td>\n",
       "      <td>3</td>\n",
       "      <td>False</td>\n",
       "      <td>med-low</td>\n",
       "      <td>538.5</td>\n",
       "      <td>56.80</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>18</th>\n",
       "      <td>VA</td>\n",
       "      <td>76</td>\n",
       "      <td>510</td>\n",
       "      <td>356-2992</td>\n",
       "      <td>no</td>\n",
       "      <td>yes</td>\n",
       "      <td>33</td>\n",
       "      <td>189.7</td>\n",
       "      <td>66</td>\n",
       "      <td>32.25</td>\n",
       "      <td>212.8</td>\n",
       "      <td>65</td>\n",
       "      <td>18.09</td>\n",
       "      <td>165.7</td>\n",
       "      <td>108</td>\n",
       "      <td>7.46</td>\n",
       "      <td>10.0</td>\n",
       "      <td>5</td>\n",
       "      <td>2.70</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>low</td>\n",
       "      <td>568.2</td>\n",
       "      <td>57.80</td>\n",
       "      <td>Normal Users</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>19</th>\n",
       "      <td>TX</td>\n",
       "      <td>73</td>\n",
       "      <td>415</td>\n",
       "      <td>373-2782</td>\n",
       "      <td>no</td>\n",
       "      <td>no</td>\n",
       "      <td>0</td>\n",
       "      <td>224.4</td>\n",
       "      <td>90</td>\n",
       "      <td>38.15</td>\n",
       "      <td>159.5</td>\n",
       "      <td>88</td>\n",
       "      <td>13.56</td>\n",
       "      <td>192.8</td>\n",
       "      <td>74</td>\n",
       "      <td>8.68</td>\n",
       "      <td>13.0</td>\n",
       "      <td>2</td>\n",
       "      <td>3.51</td>\n",
       "      <td>1</td>\n",
       "      <td>False</td>\n",
       "      <td>high</td>\n",
       "      <td>576.7</td>\n",
       "      <td>60.39</td>\n",
       "      <td>No VM plan</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   state  account length  area code phone number international plan  \\\n",
       "0     KS             128        415     382-4657                 no   \n",
       "1     OH             107        415     371-7191                 no   \n",
       "2     NJ             137        415     358-1921                 no   \n",
       "3     OH              84        408     375-9999                yes   \n",
       "4     OK              75        415     330-6626                yes   \n",
       "5     AL             118        510     391-8027                yes   \n",
       "6     MA             121        510     355-9993                 no   \n",
       "7     MO             147        415     329-9001                yes   \n",
       "8     LA             117        408     335-4719                 no   \n",
       "9     WV             141        415     330-8173                yes   \n",
       "10    IN              65        415     329-6603                 no   \n",
       "11    RI              74        415     344-9403                 no   \n",
       "12    IA             168        408     363-1107                 no   \n",
       "13    MT              95        510     394-8006                 no   \n",
       "14    IA              62        415     366-9238                 no   \n",
       "15    NY             161        415     351-7269                 no   \n",
       "16    ID              85        408     350-8884                 no   \n",
       "17    VT              93        510     386-2923                 no   \n",
       "18    VA              76        510     356-2992                 no   \n",
       "19    TX              73        415     373-2782                 no   \n",
       "\n",
       "   voice mail plan  number vmail messages  total day minutes  total day calls  \\\n",
       "0              yes                     25              265.1              110   \n",
       "1              yes                     26              161.6              123   \n",
       "2               no                      0              243.4              114   \n",
       "3               no                      0              299.4               71   \n",
       "4               no                      0              166.7              113   \n",
       "5               no                      0              223.4               98   \n",
       "6              yes                     24              218.2               88   \n",
       "7               no                      0              157.0               79   \n",
       "8               no                      0              184.5               97   \n",
       "9              yes                     37              258.6               84   \n",
       "10              no                      0              129.1              137   \n",
       "11              no                      0              187.7              127   \n",
       "12              no                      0              128.8               96   \n",
       "13              no                      0              156.6               88   \n",
       "14              no                      0              120.7               70   \n",
       "15              no                      0              332.9               67   \n",
       "16             yes                     27              196.4              139   \n",
       "17              no                      0              190.7              114   \n",
       "18             yes                     33              189.7               66   \n",
       "19              no                      0              224.4               90   \n",
       "\n",
       "    total day charge  total eve minutes  total eve calls  total eve charge  \\\n",
       "0              45.07              197.4               99             16.78   \n",
       "1              27.47              195.5              103             16.62   \n",
       "2              41.38              121.2              110             10.30   \n",
       "3              50.90               61.9               88              5.26   \n",
       "4              28.34              148.3              122             12.61   \n",
       "5              37.98              220.6              101             18.75   \n",
       "6              37.09              348.5              108             29.62   \n",
       "7              26.69              103.1               94              8.76   \n",
       "8              31.37              351.6               80             29.89   \n",
       "9              43.96              222.0              111             18.87   \n",
       "10             21.95              228.5               83             19.42   \n",
       "11             31.91              163.4              148             13.89   \n",
       "12             21.90              104.9               71              8.92   \n",
       "13             26.62              247.6               75             21.05   \n",
       "14             20.52              307.2               76             26.11   \n",
       "15             56.59              317.8               97             27.01   \n",
       "16             33.39              280.9               90             23.88   \n",
       "17             32.42              218.2              111             18.55   \n",
       "18             32.25              212.8               65             18.09   \n",
       "19             38.15              159.5               88             13.56   \n",
       "\n",
       "    total night minutes  total night calls  total night charge  \\\n",
       "0                 244.7                 91               11.01   \n",
       "1                 254.4                103               11.45   \n",
       "2                 162.6                104                7.32   \n",
       "3                 196.9                 89                8.86   \n",
       "4                 186.9                121                8.41   \n",
       "5                 203.9                118                9.18   \n",
       "6                 212.6                118                9.57   \n",
       "7                 211.8                 96                9.53   \n",
       "8                 215.8                 90                9.71   \n",
       "9                 326.4                 97               14.69   \n",
       "10                208.8                111                9.40   \n",
       "11                196.0                 94                8.82   \n",
       "12                141.1                128                6.35   \n",
       "13                192.3                115                8.65   \n",
       "14                203.0                 99                9.14   \n",
       "15                160.6                128                7.23   \n",
       "16                 89.3                 75                4.02   \n",
       "17                129.6                121                5.83   \n",
       "18                165.7                108                7.46   \n",
       "19                192.8                 74                8.68   \n",
       "\n",
       "    total intl minutes  total intl calls  total intl charge  \\\n",
       "0                 10.0                 3               2.70   \n",
       "1                 13.7                 3               3.70   \n",
       "2                 12.2                 5               3.29   \n",
       "3                  6.6                 7               1.78   \n",
       "4                 10.1                 3               2.73   \n",
       "5                  6.3                 6               1.70   \n",
       "6                  7.5                 7               2.03   \n",
       "7                  7.1                 6               1.92   \n",
       "8                  8.7                 4               2.35   \n",
       "9                 11.2                 5               3.02   \n",
       "10                12.7                 6               3.43   \n",
       "11                 9.1                 5               2.46   \n",
       "12                11.2                 2               3.02   \n",
       "13                12.3                 5               3.32   \n",
       "14                13.1                 6               3.54   \n",
       "15                 5.4                 9               1.46   \n",
       "16                13.8                 4               3.73   \n",
       "17                 8.1                 3               2.19   \n",
       "18                10.0                 5               2.70   \n",
       "19                13.0                 2               3.51   \n",
       "\n",
       "    customer service calls  churn state_churn_rate  total_dom_minutes  \\\n",
       "0                        1  False              med              707.2   \n",
       "1                        1  False          med-low              611.5   \n",
       "2                        0  False             high              527.2   \n",
       "3                        2  False          med-low              558.2   \n",
       "4                        3  False          med-low              501.9   \n",
       "5                        0  False          med-low              647.9   \n",
       "6                        3  False              med              779.3   \n",
       "7                        0  False          med-low              471.9   \n",
       "8                        1  False              low              751.9   \n",
       "9                        0  False              low              807.0   \n",
       "10                       4   True          med-low              566.4   \n",
       "11                       0  False              low              547.1   \n",
       "12                       1  False              low              374.8   \n",
       "13                       3  False             high              596.5   \n",
       "14                       4  False              low              630.9   \n",
       "15                       4   True              med              811.3   \n",
       "16                       1  False          med-low              566.6   \n",
       "17                       3  False          med-low              538.5   \n",
       "18                       1  False              low              568.2   \n",
       "19                       1  False             high              576.7   \n",
       "\n",
       "    total_dom_charges vmail_messages  \n",
       "0               72.86   Normal Users  \n",
       "1               55.54   Normal Users  \n",
       "2               59.00     No VM plan  \n",
       "3               65.02     No VM plan  \n",
       "4               49.36     No VM plan  \n",
       "5               65.91     No VM plan  \n",
       "6               76.28   Normal Users  \n",
       "7               44.98     No VM plan  \n",
       "8               70.97     No VM plan  \n",
       "9               77.52   Normal Users  \n",
       "10              50.77     No VM plan  \n",
       "11              54.62     No VM plan  \n",
       "12              37.17     No VM plan  \n",
       "13              56.32     No VM plan  \n",
       "14              55.77     No VM plan  \n",
       "15              90.83     No VM plan  \n",
       "16              61.29   Normal Users  \n",
       "17              56.80     No VM plan  \n",
       "18              57.80   Normal Users  \n",
       "19              60.39     No VM plan  "
      ]
     },
     "execution_count": 29,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df['vmail_messages'] = pd.cut(df['number vmail messages'],bins=[0,1,38,52],\n",
    "                             labels=['No VM plan','Normal Users','High Frequency users'],\n",
    "                             include_lowest=True)\n",
    "df.head(20)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "184ae590",
   "metadata": {},
   "source": [
    "## Feature Selection"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "fa80dcc9",
   "metadata": {},
   "source": [
    "### Correlation Analysis\n",
    "\n",
    "In order to visualize and investigate the pair-wise correlations between the features and churn, we'll use a bar graph and heatmap."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 30,
   "id": "4d07c7da",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<AxesSubplot:>"
      ]
     },
     "execution_count": 30,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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DBQMAQAAAAAAABgoGAIAAAAAAAAwUDAEAAAAAAAAYKBgCAAAAAAAAMBgu3Tpkt3sEAAAAAAAAAAyBkYYAgAAAAAAADBQMAQAAAAAAABgoGAIAAAAAAAAwEDBEAAAAAAAAICBgiGQhmPHjikpKcnsGKazcj9cuHBBu3fv1p9//ml2FAAAAACZhBXPsVJSUpSSkmLcPnPmjObOnavt27ebmCpjuHXrltkR7omCISxv2LBhmjdvniTJbrerWbNmqlKlikqXLq3du3ebnM596AeHP/74Q126dJG/v7/q16+vhIQESdIHH3ygUaNGmZzOffhQd1i6dKnWrVtn3I6IiFBAQIBatGihxMREE5O514wZM/Tcc8+pSJEiOn78uCTp3//+t5YuXWpuMDdbvXq12rRpo+rVqys+Pl6SNHfuXG3YsMHkZO6zefNmp8+Eb775Rq+++qref/99Xb161cRk7nf27FlNmjRJvXv31oULFyRJ27dvN14jVnD+/HmdP3/euH3w4EGNGDFCixYtMjGV+/G6cEhKStKnn36q5s2bq1atWqpRo4bTH6vgdfF/Dh48qL59+6ply5bG96aVK1cqOjra5GTuwzmWQ+vWrfX5559Lkq5evao6deooNDRUr7/+uubPn29yOvf57LPPtGzZMuN2z5495eXlpapVqyomJsbEZGmzdMFwyZIleu+999S+fXu1bdvW6Y8VcCLssHDhQpUqVUqS42TwwIEDWrNmjdq2bauPPvrI3HBuRD84fPTRR0pISNCGDRuUK1cuo71BgwZauXKlicnciw91h9GjRxs/R0VFafz48erWrZtu3bqlIUOGmJjMfaZOnapx48apc+fOstvtRnuRIkU0ffp0E5O518KFCxUUFCQ/Pz+dOHFCycnJkqTbt29rwoQJJqdzn4EDB+rMmTOSpJiYGH3wwQcqV66cdu7cqbCwMJPTuU9UVJSqVq2qhQsX6quvvtIff/whSVq/fr1GjBhhcjr36dKli3788UdJjpH5r732mlauXKnevXtr0qRJJqdzH14XDn369NG///1vFS9eXI0aNVKTJk2c/lgFrwuHdevWqW7dujp9+rQ2btxojKiLjY1VRESEyench3Msh6ioKL344ouSpBUrVujxxx/X0aNHNWHCBEu9Lj7//HMVLFhQkrRlyxZ99913mjFjhipUqJAhzy0sWzAMDQ3V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TUyZ/sqUKaOtW7e6XGxcunRppt8sjTUM/090dLTsdrsqVaqkdevWOS35lD17dhUqVCjT74jLGoauPv/8cy1btsxpYE6ePHnUt29fNW3aVB9++KGJ6dIX55oO06dPf6jjbDabZQqGr732mkaPHq05c+YYbSdOnNDQoUPVuHFjE5OlzbIFw8y86O7DsvKJMOtKuBoyZIh69OihChUqyG63q3r16rLb7WrZsmWm/kC/m4+Pj2bPnq3hw4c7tc+ZM8dYZ+LChQuZfhRyz5491a9fP924cUN2u107d+7UggULNHHixP/pauE/3ciRI9W4cWNVrVpVSUlJeuutt3Ts2DEVLlxYs2fPNjueW+TKlUtbt26Vn5+fU/vWrVuN9dlu376tnDlzmhHPbex2u3777TeX3R0PHTpknBBky5Yt069PFR4erkaNGmnv3r36888/NWTIEB06dEhXrlzRqlWrzI7nNufOndOff/7p0n7z5k1jSl6hQoV048YNd0dzq/79+6tbt246deqUbt++re+++05HjhzRokWLtHDhQrPjpSvWMPw/qRcXrbSEzd1Yw9DVtWvXlJiYqDJlyji1nzlzJtO/N3Ku6cAahq6GDx+u1q1by9/fX9evX1fDhg119uxZVa9eXUOGDDE7ngtL7pIMh4SEBKOKffz4cQUGBhonwj/88IOldoPF/zl+/Liio6OVkpKiwMBAl5PjzG7VqlXq1KmTSpYsqcqVK8tms2nfvn2KjY3V3LlzVb9+fc2YMUPHjh3Txx9/bHbcdDVnzhyNHTtWp06dkiQVLVpU/fv3V6dOnUxO5l43btzQokWLtH//fqWkpKhixYpq1apVpt/MINX48eM1ZswYvfnmm8ZrYu/evZo3b5769u2rDz74QJMnT9aaNWv03XffmR033QwcOFALFizQBx984NQPn376qdq2bauPP/5Yc+bM0YIFC/Tjjz+aHTddnTlzRl9++aXxWVGxYkUFBwfLy8vL7Ghu07ZtW506dUoTJkwwppNFRUXp/fffV7FixTR//nz98MMPGjFiRKYfXbZ27Vp98sknTs+Hfv36qW7dumZHg5vda4T1nZsBZfaRp/g/77zzjjZu3Khhw4apatWqkqTdu3dr6NCheuGFFzRt2jSTEwLm2bBhg9O5xUsvvWR2pDRZtmD4oHUFMvNaAney+onwnRISEhQfH6+bN286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RHnz5tXvv/8uPz8/VatWTe+++67J6dzn7g2yUp8PM2bMsMz6ZJLjvfHAgQMqUaKEU3t0dLSlvmO/8cYbWrNmjWWmYN9Lzpw5lZiYKF9fX6f206dPW2oDnGnTpmnSpEku7cWLF1dISIhlniecazrUrFlT48aN05w5c4zNPpKTkzVu3LgHbjKYmXz88cfq37+/evfuLW9vb33++efy8vJSt27dMuQu6pYtGFavXl0xMTFpbvVuFZMnT9aJEydUtmxZFS1a1GW0jFXWperbt686dOig2NhYvfDCC5Icm1589913+vrrr01O5z53L0SdunN41apV5enpaU4oEzz//PNat26dypUrp+bNm6t///5av369Nm7caGx1bwWpa5nefcWvWrVqGW6ofHrKnz+/020PDw+VLVtWYWFhGWLnMneoWLGi9u7dK39/f9WqVUsjRozQ2bNntXDhQpUrV87seG5TtmxZHThwQL6+vqpSpYomTJigLFmyaO7cuZb6LvHrr7863U79rPj444/15ptvmpTK/Xx9fXX8+HH5+Pjo6aef1uLFi1WlShWtWLHinptGZUadO3d2um2z2VSwYEG9+OKLGjFihEmp3K9Vq1YaMGCA8uTJY0xL3rRpkwYNGmSpTeOKFSumUaNGaceOHSpXrpzLqCqr7Jr98ssvKzw8XPPnzze+Q1+8eFHDhg3Tyy+/bG44N2LkrQPnmg7Dhg3Tq6++qsqVKxtroW/fvl3Xrl3TDz/8YHI69zl69KhatGghScqaNauuX7+unDlzql+/fmrTpk2Ge5+01BqGd46YOXnypEaOHKmQkBAFBAS4fKBVqlTJveFM8KCpZAMGDHBTEvOtWbNG48aN0/79+yU5dv3r06ePXnnlFZOTwd0uXryopKQkFSlSRCkpKZo4caK2b98uf39/ffjhh5Ypnt69jkhqUeDuHd6Q+e3bt09//PGHXnzxRZ0/f17du3fXjh079NRTT2nKlCmWKRquXbtW165dU5MmTXT8+HG1adNGR44cUYECBTRr1izjJADWMGXKFGXJkkXdu3fXhg0b1LZtW926dUspKSkaPXq05ddws5pbt26pe/fuWrJkiTFyJiUlRc2aNdPnn39umVFl95uBYLPZLLNrdmJiol577TWdP3/e+Iw8ePCgChYsqO+//94yM9wqVKigUaNGuazluHz5cg0cOFAHDx40KZn7ca7pkJiYqC+++EIHDhyQ3W5XxYoVLbcJTunSpbVs2TKVKVNGzz33nIYMGaLXX39d0dHRatSokeLj482O6MRSBcN8+fKlOWLmblbY9AS4l4SEhDR3DrdCER0A/hcXL16Up6en03IOsKa4uDjt27dPTz31lGWK6HB17Ngx7d+/3zgRvntxf1jH9evXFRkZ6VQYadmypWXWP5YcI8oWLlzotCHQpk2b9O6776ply5aW30gQ1tS+fXvVr19fXbp0UVhYmJYvX662bdtq5cqVKlSokJYuXWp2RCeWKhj+LzvvFC9ePB2TABlPdHS0unXrpiNHjrgU1a1YRKdwKsXHx2vbtm1p9kNGGy6fXi5evKjhw4drw4YNOnfunMtrIy4uzqRkgDmSkpL02WefGa+Ju98brLKcCf7P7t277/l8uHu5E1jH2bNnVbBgQXl4WHaPTctj5C3g6vjx47p69arKly+v69eva8iQIcZstpEjR6Y5jd9MlioY3mn48OEqVqyY3nrrLaf2mTNn6vTp0xoyZIhJydzn5s2bGjdunBYvXqz4+HjdunXL6X6rFYisrk6dOsqfP7/69esnLy8vlxEzVimiUzh1WLhwoXr27KmsWbOqQIECTs8HK00r6tChg/bv368uXbqk+bpo3769Scnch6KpA4Uyh5CQEK1cuVLNmjVL8zVhpeVMKJRJkyZNUlhYmPz8/FyeDzabTStWrDAxHdzt1q1bGj58uGbOnKkbN25oz5498vX11dChQ+Xj42OZTS7gjJG3wD+XZTc9+fbbbzV79myX9kqVKmn8+PGWKBiOHDlSS5YsUe/evTVo0CANGzZMJ0+e1JIlSzR48GCz48HNDh8+rI0bN8rf39/sKKZ6//33VaxYMU2YMCHNk2Gr+Pjjj9WzZ08NHjzYuCpsRRs3btTSpUtVtWpVs6OYpmfPnvctmlpFnz59jELZs88+a9l++P777zVnzhxLbQKVlgcVyqzis88+U0REBGs2QpIUERGhn376SZ9//rn+9a9/Ge3PPPOMJkyYQMHQovz8/CgSAv/f+fPnJUkFCxaU5FjfdOnSpSpTpoxatmxpZrQ0WbZgeO7cOeM/6U758+fXuXPnTEjkfkuXLtW///1v1atXT6GhoWrUqJFKliyp0qVLa/369QoKCjI7ItwoICBAZ86csXzBkMKpw7lz59SpUydLFwslx4d5njx5zI5hKoqmDhTKHHLnzq1ixYqZHcN0FMoc/vjjD9WvX9/sGMggFi1apMmTJ6tWrVpOU5EDAgJ09OhRE5MBQMbQpUsXtWnTRh07dtSFCxf02muvqUiRIpo+fboSEhLUq1cvsyM6seyiEt7e3mlOH9qyZYuKFi1qQiL3O3funEqXLi1JypMnjy5fvixJevnll7V+/Xozo8EEoaGhGjp0qH755RedPXtWFy9edPpjFamFU6t75ZVXtHv3brNjmC40NFQff/yxrl69anYU01A0daBQ5vDuu+9qypQpLlNwrYZCmcMbb7yhNWvWmB0DGURiYmKa628lJyfr9u3bJiQCzDd//nz9+eefLu03b97U/PnzTUgEMx08eFDVqlWTJC1btkx+fn7avn27pk2bluYMWLNZdoRhly5dNGjQIN26dUsvvviiJGnDhg0KDw/X+++/b244N/H29jY+2P38/LR27VpVqlRJu3btUs6cOc2O51YzZszQjBkzdOLECW3btk2+vr7697//LV9fXzVv3tzseG7RrFkzSVLz5s2dplPZ7fZMv3bfnQXR1MLpkCFDFBAQ4LIgc758+dwdzxR16tTRRx99pEOHDikgIEBZszp/XDRp0sSkZO41btw4nTx5UqVKlZKPj49LP1hh3brUoum0adP02GOPmR3HNKmFsvHjx1t6Ef/169dr27ZtWrNmjcqUKePymliwYIFJydwrtVBm9SmWxYoV06hRo7Rjxw6VK1fO5flglQ2yatWqpU6dOql169by9PQ0O45pypQpo61bt6pEiRJO7UuXLlXFihVNSmWeffv2KTY2Vg0aNFCePHl07do15ciRw+V1gswtJCRE9erVU6FChZzar169qpCQELVr186kZOmvbdu2D32sVb4/JCUlGRfif/nlFzVs2FCSVLFiRZ06dcrMaGmy7LtVr1699Pvvv6t///66efOmJCl79uzq3r273nvvPZPTucfrr7+uDRs2qFq1aurevbu6du2qOXPmKCEhQe+++67Z8dxm6tSpmjhxot577z2Fh4cb7alDg61SMLTywuR+fn4uRVIrFk7vlHrh5JNPPnG5z0r9YJXC6N1q1KjhdNuqRdO7v+hu3brV8oWyAgUK6PXXXzc7hikmT55s/EyhzGHu3LnKkyePduzYoR07djjdZ7PZLNMPDRo00MSJExUWFqZGjRqpU6dOql27ttmx3K5///7q1q2bTp06pdu3b+u7777TkSNHtGjRIi1cuNDseG5z9uxZtWvXTnv37pXNZtPevXuVJ08eDR48WDly5FBERITZEeFGqecQd4uLi1PevHlNSOQ++fPnNztChuPn56cVK1aoSZMmWr9+vVF3OXfunJ544gmT07my7C7Jqa5du6bDhw/LbrerdOnSlh49sWvXLu3YsUP+/v569dVXzY7jNtWqVdOIESPUoEEDeXt7a/PmzfL19dVvv/2m1157TbGxsWZHRDrbvHnzQx9bq1atdEwCZAyjR49+6GMz8664PXr0eOhjp06dmo5JkBEEBgY+1HFW2kke/8dut2vNmjX65ptv9OOPP6pw4cJ688031b59+zSn6WZWa9eu1SeffKLo6GilpKSoYsWK6tevn+rWrWt2NLcJDg7WtWvXNG3aNJUvX944t/jll1/Ur18/7dy50+yIbnfmzBljkE6qzP66SL34eujQIZUqVcppXfCUlBTFxcXplVdeyZDTUJF+li9fruDgYCUnJ6t27dpaunSpJMesph07digyMtLkhM4sXzAEvLy8tHPnThUvXtypYHj06FG98MILSkhIMDtiuomKilJgYKA8PDwUFRV132MrVarklkwAAAD/ZBcvXtSsWbMUERFhnBT26NFD9erVMzsa3KBUqVJatmyZAgICnM4tjh8/rho1auj06dNmR3SLy5cvq3///vruu+9cioWSMv1sldSLrxEREerZs6fTetDZs2dX8eLF1aRJE2XPnt2siG4VEhKi0aNH6/HHH3dqv3btmvr166cpU6aYlMz9zp49q4SEBFWoUMFY5mb37t3Kmzevnn76aZPTObPslGQ4xMfHa9u2bTp37pzLAuZWmUbi6+ur6OhoFS9e3Kn9559/NjaFyazq1KmjI0eOqFChQqpTp45sNpvsdtdrCFaagjp9+nQ98cQTatOmjVP7t99+qz/++CNTr1c1efJkBQcHK2fOnE5T79KSmd8ffHx8FBUVpQIFCsjb2zvNaSSp4uLi3JjMHKkjcO8eXbt582bZbDbVrFnTjFhu99tvv+n27dsqX768U/t//vMfZc2aVWXKlDEpWfqrUaOGfvjhB3l6erpMV79bZp6ifqebN28qJSXFZc3npKQkeXh4ZOoTwH79+mno0KHKkyeP+vXrd99jx4wZ46ZUGceuXbv0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5zd+HjlypEaPHu20O7Svr68KFSqksLAwNWjQwIyIbscFBYcuXbqoS5cuOnXqlCIjIxUZGanRo0eratWqatOmjYKDg82OmO527NihNWvWuEzN9vb2VmJiokmp3KNz5846cuSIChUqZKxhmBYrTSn65Zdf1LVr1zR/Xyv1g4+Pj2bPnq3hw4c7tc+ZM8f4LnHhwoVMP2V/1KhRCg8P1yeffCI/Pz+z45jGyhdd7/bnn39q4cKFOnz4sGw2m8qUKaOWLVsqR44cZkdzG54PgKsXX3xRP/74oypVqqSOHTtq0KBBWrp0qfbv369mzZqZHQ/3QMHQwoYMGaLBgwdryJAhCggIcNn9NbN/yU116NAhvfLKKy7tnp6eunjxogmJzHH69Gk988wzkqScOXPqypUrkqSWLVuqbt26lllX4vDhwypatKhLe5EiRRQTE2NCInNwQcFZsWLF9P777+v9999XVFSUevXqpX79+lmiYCjJaTODVPHx8cqbN68Jadznzs8AK30e3M+AAQNUv359hYWFWWrNwruNGDFCnTp10urVq1W5cmXZbDbt27dPsbGxmjt3riRp7969mfIk6O516pKSklS1alXlyJHDZTH/zH7BMZWVL7re6dChQ2rZsqWuXLlijMCdM2eORo0apcWLF6t06dImJ3QPng+AqwkTJhizEt566y15enpq+/btatKkiYKCgkxOh3uhYGhhrVq1kuTY6v3uBYqtNErA09NTCQkJTlPtJMd0gbQKR5kVw8QdypQpo4iICE2ZMkW5cuWSJN24cUNjxoxRmTJlTE7nPlxQcLVt2zZFRkbqu+++061bt9S6dWuzI7lF3bp1NWXKFE2ePNlou3LlikaNGqX69eubmAxmOHnypObPn2/pYqEkNWjQQLt379bMmTMVExMju92uhg0bKigoSD4+PpKUaS8ojBkzxuwIGQ4XXR0GDBigChUq6PPPPzcuKF25ckVvv/22Bg4cqCVLlpic0D14PgCuTp065TSbr0WLFmrRooXsdrvi4+ONz05kLBQMLWzFihVmR8gQWrZsqbCwMM2aNUs2m03JycnavHmzQkND1aFDB7PjuQ3DxB3Gjx+vNm3aqGzZssbV8V9//VVZsmTRwoULTU7nPlxQcPjtt98UGRmpRYsWKSEhQS+99JIiIiL0+uuvGwXlzG7kyJFq3LixqlatqqSkJL311ls6duyYChcurNmzZ5sdD25WvXp1xcTEWGodz3vx8fHR0KFDzY7hdqxT54qLrg47duzQunXrnEaf582bV6GhoWnO5smseD4AripWrKjDhw+rUKFCTu0XL15UxYoVM/W5xZ0X3R+kZ8+e6Zjkf0fB0MJq1apldoQMYciQIerRo4cqVKggu92u6tWry263q2XLlvrwww/Njuc2DBN3eOaZZxQdHa2FCxfqyJEjstvtat26tVq2bKk8efKYHc9tuKDgUKNGDT3zzDPq0aOHWrZsqYIFC5odye2KFCmiTZs2adGiRdq/f79SUlLUpUsXtWrVyjJFU/yfoKAghYaGKjExUQEBAS5TUCtVqmROMDeIiopSYGCgPDw8FBUVdd9jM3M/3Ole0yttNpty5sxpmfdMLro65MiRQ5cvX3Zpv3LliqXWMOT5ALhKHXRwt6tXrypnzpwmJHKf6dOnP9RxNpstwxUMbZcuXbKbHQLmOXv2rL744gunhYm7du1qyV1Rjx8/rujoaKWkpCgwMFBPPfWU2ZEAmOy///0v7wXAHe63HEFmH32cL18+YxOcfPnyyWazyW53/Rqd2fvhTqn9cC+PP/64OnTooGHDhrkUlzOTlJQUpaSkGL/jkiVLtH37dvn7+ysoKMhlWY/Mqnv37tq3b58mTJigatWqSZJ27typDz74QM8884ymTp1qckL34PmAGjVqPPSxW7duTcck5uvXr58kacaMGerQoYPTxeaUlBTt2bNH2bNn16pVq8yKiPugYGhh27dvV8uWLVWoUCHjQ33Xrl06f/68Fi9erGeffdbkhOY5duyYihYtmumvdmzZsuWhj61Zs2Y6JkFGxAUFh6SkJK1atUqxsbHq0qWLPD09FRsbK09PT0uu5QhrO3ny5H3vL168uJuSuN/Jkyfl4+Mjm81m6X6405IlSxQWFqa33npLVapUkSTt2bNHs2fP1oABA3T58mWNGzdOXbt21aBBg0xOm37i4uJcNoORZLm1uS5duqR33nlHP/30k7JkySLJURBo2LChpk6dqieeeMLkhO7B8wGjR49+6GMHDBiQjknM9/rrr0tynHc+++yzTgXz7Nmzq3jx4urVqxcX6DMoCoYW9sorryggIED//ve/5eHhIcnxof7BBx/ot99+088//2xyQvcYNmyY/P391b59e9ntdjVv3lwbNmxQ3rx5tXjxYlWtWtXsiOnm7hESqV9s7r4tyTKjJeDABQWHY8eOqWnTprp27ZouX76sPXv2yNfXV0OGDNHly5c1adIksyMCgGkaNWqkbt26qUmTJk7ty5cv12effaYffvhBixYt0qhRo7Rnzx6TUqa//Pnzp7k21++//y5/f3/LfYc6duyYDh8+LLvdrjJlysjPz8/sSG7F8wFw1aNHD40ePdppjVOrmDx5soKDg5UzZ84HrmfIlGRkGF5eXtq0aZNKlSrl1H7kyBG9+OKLSkxMNCmZe5UvX16zZs1StWrV9PPPP+udd97RwoULtXDhQh08eFArV640O2K6ufMLy+7duxUaGqo+ffoYxaCdO3dq/PjxCg8PV4MGDcyKCRNwQcGhTZs28vLy0vjx41WiRAlt3rxZvr6+2rJli0JCQh64jhn+2ZhS5LB8+XI1bNhQ2bJl0/Lly+977N2Fo8zs+vXrOnDggM6dO2esAZzKKv3g5eWlLVu2uIwMOXr0qF544QUlJCToxIkTeu6555SQkGBSyvSXL18+xcTEuKzZePLkST333HM6ffq0Scnc6+bNm0pJSXGZoZOUlCQPDw9lz57dpGTuxfMBuLekpCRjA6CSJUtm+hl9khQYGKhffvlF+fPnV2Bg4D2Ps9lsio6OdmOyB8u8i4nggfLmzasTJ064FAxPnDhhmSkDknTu3DkVLVpUkrR69Wo1b95cVapUUb58+fTSSy+ZGy6d5c+f3/h55MiRGj16tOrUqWO0+fr6qlChQgoLC6NgaDEHDhzQ1KlTjWKhJHl4eCgkJEQvvviiicnca8eOHVqzZo0xtSqVt7e3ZS6qWJlVij4P0rlzZ2Ptvs6dO9/zOCut3ffLL7+oa9euaf6+VuoHHx8fzZ49W8OHD3dqnzNnjry9vSVJFy5cyLTLN6SuzWWz2RQeHp7m2lwVKlQwK57bde7cWTVr1nQZITNz5kxt3rxZ8+bNMymZe/B8wL18/fXXWrx4seLj43Xz5k2n+zJagSi9JCcnKzw8XF988YVu3rwpu92uHDly6O2331ZoaGimXttz//79af78T0DB0MJatGihXr16KTw8XM8++6xsNpu2b9+u8PBwvfHGG2bHc5v8+fMrLi5OxYoV07p16xQWFibJ8aZmJYcPHzYKp3cqUqSIYmJiTEgEM3FB4f/cunXLpS0+Pt6SUyqsJrOvK/SwLl68mObPVjZgwADVr19fYWFhKlKkiNlxTDNixAh16tRJq1evVuXKlWWz2bRv3z7FxsZq7ty5kqS9e/dm2p1hf/31V0mOpVyOHDnisjZXxYoV1atXL7Piud2OHTsUGhrq0l6nTh2NHz/ehETuxfMBaZk4caLGjx+voKAgbd26VV27dtWxY8e0detWSz0fwsLCtHjxYo0fP17PP/+8JMfsjGHDhiklJUUjRowwOSHSQsHQwoYNGya73a6ePXsaxbFs2bLprbfe0kcffWRuODdq3LixgoOD5e/vr4sXL6pevXqSHCOsSpYsaXI69ylTpowiIiI0ZcoU44rojRs3NGbMGJUpU8bkdOmLaYeuuKDgULduXU2ZMsVpvZErV65o1KhRql+/vonJ0hevCeD+Tp48qfnz51u6WChJDRo00O7duzVz5kzFxMTIbrerYcOGCgoKMjZ2CA4ONjll+kldtsbKa3Pd6caNG2nuhu3h4aGrV6+akMi9eD4gLXPmzNGECRPUtGlTffHFF3r77bfl6+urMWPGKC4uzux4brNo0SJNnjzZ6ftzyZIlVbBgQb377ruWKhhevHhRa9asSXPEaf/+/U1KlTbWMISuX7+u2NhY2e12+fn5KXfu3GZHcqvk5GRNmzZN8fHxat++vSpWrChJmjJlih5//HF16tTJ5ITusXfvXrVp00a3bt1SuXLlJDmulGbJkkULFy7UM888Y3LC9MNOZq5u3ryp0NBQzZo1y+WCQnh4uGXWIUpISFDjxo0lScePH1dgYKCOHTumwoUL64cffnBZnyiz4DWRNqYUIVXz5s31zjvvZOoLB/jfWXFtrjvVq1dPdevWddkRe8SIEVq7dq3Wr19vUjJzWP35AIciRYpo586d8vHxkb+/v5YsWWJ8n6xbt66OHz9udkS3YP8Eh127dql169bKkSOHzp8/ryJFiujMmTPKkSOHfHx8MtyFeAqGAAzXr1/XwoULdeTIEWNnu5YtWypPnjxmR4NJrH5BQXKMmFi0aJH279+vlJQUVaxYUa1atXJamwiZ351TiqZOneoypahv375mR4QbLV++XCNHjlRISIgCAgJcRlVVqlTJnGBuEBUVpcDAQHl4eDxw46fM3A93svLaXHdatWqVOnTooObNm+uFF16QJG3cuFHfffedvv76a7366qsmJ3QPng+4U8WKFTVnzhxVqlRJderU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\n",
      "text/plain": [
       "<Figure size 1440x360 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Show correlation plot for correlation of Churn with each of the remaining features.\n",
    "plt.figure(figsize=(16,10))\n",
    "df.corr()['churn'].sort_values(ascending=False).plot(kind='bar', figsize=(20,5))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 31,
   "id": "c381e7ba",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 1440x1080 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "cor = df.corr()\n",
    "plt.figure(figsize=(20,15))\n",
    "sns.heatmap(cor.round(3),annot=True,cmap='inferno')\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "174f73da",
   "metadata": {},
   "source": [
    "I'm going to remove highly correlated features:\n",
    "\n",
    "* total_night_minutes\n",
    "* total_eve_minutes\n",
    "* total_day_minutes\n",
    "* tatal_intl_minutes"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 32,
   "id": "7227b680",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Generate box plots using the math library\n",
    "\n",
    "import math\n",
    "\n",
    "def boxplot(x, y, df):\n",
    "    columns = len(y)\n",
    "    rows = math.ceil(columns/3)\n",
    "    plt.figure(figsize=(7*columns, 7*rows))\n",
    "    for i, j in enumerate(y):\n",
    "        plt.subplot(rows, columns, i+1)\n",
    "        ax = sns.boxplot(x=x, y=j, data=df[[x, j]], palette=\"Blues\", linewidth=1)\n",
    "        ax.set_title(j)\n",
    "    return plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 33,
   "id": "b4fde946",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<Figure size 6000x4500 with 0 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "iVBORw0KGgoAAAANSUhEUgAABV0AAAHjCAYAAAAwkb8wAAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjUuMSwgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/YYfK9AAAACXBIWXMAAAsTAAALEwEAmpwYAACWzUlEQVR4nOzdeVhU5f//8deAC2ooqIgL4opr7uaCCy4fVzRFMU0/muZullqakqZF7kKlhmhpZSaVKZbkXi6568dyadFQc8kFBQFXSGB+f/hjvhGgjg4clufjurwuz7nvc+Y1bGfOe+65b1NMTIxZAAAAAAAAAACbsDM6AAAAAAAAAADkJBRdAQAAAAAAAMCGKLoCAAAAAAAAgA1RdAUAAAAAAAAAG6LoCgAAAAAAAAA2RNEVAAAAAAAAAGyIoitypZEjR8rJyUnnzp3LsHPv2rXL5ufOambNmiUnJyetXLnS6CgAAGR7586dk5OTk0aOHGl0lAyX3nPNyNdoAGAU7j8zT61atVSrVi2jYwCSKLoig+zatcumNw1OTk784QQA4BF5e3tTuAIA5BrcfyIr4PUX/i2P0QEAAAAASKVLl9bBgwdVuHBho6MAAJAtrVu3zugIgAVFVwAAACALyJs3r6pUqWJ0DAAAsq0KFSoYHQGwYHoB2NysWbPUtWtXSdIXX3whJycny7/kuT/NZrM+/fRTtW3bVm5ubipVqpSaN2+uhQsX6u+//7acK/ljIpJ04cKFFOf650dHvvvuOw0dOlT169dX6dKlVaZMGbVs2VKLFi1SYmJihjzPHTt2qFOnTipdurTKly+vvn376uTJkw885scff1SvXr1UoUIFlShRQnXq1NHEiRN17dq1VH3/OTfP6tWr5eXlpVKlSqlatWp64403FB8fL0navn27OnfuLDc3N7m7u2vYsGG6fv36Ez23uLg4LVy4UG3atFHZsmVVqlQp1a9fXy+//LJOnz6d7nPz9vaWm5ubypYtq169eun3339P1e/UqVN666231KpVK1WqVEklSpTQ008/rVdeeUUXLlxI1f+fHxU6ceKE/vvf/6pixYpycnLSsWPHLHlnzpyp2rVrq0SJEqpdu7amT5+u+Pj4B3406Ntvv1W3bt1Uvnx5lShRQvXr19dbb72lGzdupOp79OhRDR48WLVq1ZKrq6sqVqwoT09Pvfbaa4qNjbXmywsghzty5IiGDBmimjVrqkSJEvLw8FCnTp20bNkyS5+Hzd2Z3txvYWFhevbZZ1W1alWVKFFCVatWVYcOHRQYGGjp4+TkpD179kiS6tSpY7lu/vtv4ZkzZzRq1CjVqFFDLi4u8vDw0MCBA3X8+PFUeVauXCknJyfNmjVLP//8s3r27Cl3d3e5u7urf//++uuvvyznHDhwoCpVqqSSJUvK29s7zfNJ/3et8fLyUpkyZVS6dGm1atVKH3/8scxmc4q+yV8vb29vXbp0SSNGjFCVKlVUtGhRfffdd+l9KyRJ8fHxWrRokVq2bKny5curZMmSevrpp+Xr65vmaJiIiAhNmjRJ9evXl6urq8qVKycfHx/t3LnzgV+XAwcOqEePHipXrpycnJx06dIlFS1aVJ6enulme/HFF+Xk5GQ594N+Lqy5NiclJemzzz5Thw4d5O7uLldXVzVt2lTvvvtuitdZj+JRH/fy5cuaPXu22rdvrypVqsjFxUXVqlXT4MGD03w9YK1H+dkHkDtx/5m+7HD/aevHTWtO139er48dO6bnnntO7u7uKlWqlDp16qT9+/enmyutqQLSul4/6uuv2NhYzZgxQ02bNlWpUqXk5uamjh076ptvvkn1OGazWZ9//rnat2+vSpUqydXVVTVq1FDXrl21fPnyh39xYThGusLmmjdvrvPnz+uLL77Q008/LW9vb0tb8h+cYcOG6euvv1bp0qXVt29f5c2bV5s2bdKbb76p77//XmvWrFGePHnk7u6uiRMnas6cOSpcuHCKP2r//OP19ttvy87OTg0bNlTp0qUVGxurnTt36o033tBPP/2kpUuX2vQ5fvvttxo0aJDy5s2r7t27q3Tp0tq/f7/atWunp59+Os1jPvnkE7366qsqUKCAunXrppIlS+rAgQNasmSJ1q9fr40bN6ps2bKpjvvwww8tFxhPT09t2rRJixYt0vXr19WpUycNHz5cHTt21AsvvKCdO3dq1apVun79ulavXv1Yzy0mJkbdunXT0aNHVaFCBfXp00cFCxbUuXPnFBYWpiZNmqhSpUopjtm8ebM2btyo//znPxo0aJBOnjypLVu26KefftKBAwdUvHhxS9+wsDB9/PHHatGihRo1aqR8+fLp999/14oVK7Rx40bt2LFDZcqUSZXrzz//VPv27VW1alX16dNHsbGxKliwoMxms/773//q+++/V8WKFTV06FAlJCToiy++eOBN3muvvaZly5apTJky6tKli5ycnPS///1P77//vrZs2aLNmzfL0dFRknTs2DG1b99eJpNJHTt2VIUKFXTr1i2dP39eISEheumll1SkSJHH+noDyFlWrFihcePGSZLlb1Z0dLR++eUXzZ8/X4MHD37scy9btkyvvfaaSpQooQ4dOsjFxUVRUVE6efKkPvnkE7322muSpIkTJyokJEQXLlzQiBEjLH+f/vl36ueff1a3bt1048YNdejQQTVr1tSff/6psLAwbdy4UZ9//rnatWuXKsPPP/+sBQsWqGXLlhowYIAOHz6ssLAw/fbbb1q5cqU6duyoWrVq6fnnn9fJkye1detW+fj46MiRI3rqqacs57l586a6d++uw4cPq3bt2urbt68k6YcfftCrr76qQ4cOKTg4ONXjR0dHq3379ipcuLC6d++uhIQEOTs7P/DrNmLECK1du1bVqlXTc889p0KFCuny5cv66aef9N133+nZZ5+19P3111/l4+Oja9euqU2bNurcubOuX7+u9evXq3v37lqwYIH69++f6jEOHjyod999V56enhowYIAuX74sR0dHtW7dWj/88IOOHDmiunXrpjgmNjZWGzZskJubm1q0aPHA52DNtTkhIUH//e9/tWnTJlWuXFk9e/ZU/vz5tWfPHvn7+2vnzp2W11kPY83j7t27V/Pnz1eLFi307LPPqmDBgjp9+rS+/fZbbdy4UZs2bVLt2rUf+phpedSffQC5E/ef2fv+MzMf98iRI1qwYIEaN26sAQMG6K+//tK6devUrVs3/fjjj6patepj53+U11+XLl1S165ddfr0aTVt2lQDBw7UnTt3tGXLFg0cOFATJ06Un5+fpf9bb72l+fPny93dXd27d1eRIkUUERGhX375RV9++aVeeOGFx86LzEHRFTaXfOPwxRdfqFatWin+aEjS6tWr9fXXX6tmzZrauHGjZd6yadOmydfXVzt37tSiRYv0yiuvqFy5cvLz89OcOXNUpEiRVOdKtmrVqlQfI0hKStKIESO0atUqDR8+XM8884xNnt+tW7c0duxYmUwmrV+/Xg0bNrS0vfnmm1q4cGGqY86fP6+JEyeqYMGC+v7771W9enVL2/Tp0xUQEKDXXntNq1atSnXsrl279OOPP6pixYqSpEmTJqlBgwb66quvtGXLFq1fv14NGjSQJP39999q1aqVvv/+ex07duyxbm4mTJigo0ePqk+fPvrggw9S3JTFxcXp1q1bqY5Zv369vvnmmxQ3jW+//bbee+89ff755xo7dqxlf+/evTVq1Cjlz58/xTm2bt2q3r17KyAgQO+9916qx9i/f79effVVTZ06NcX+kJAQff/992rcuLG+/fZbOTg4SJImT56cZsFAkr766istW7ZMXbp00UcffaQCBQpY2ubNm6cZM2Zo1qxZmjlzpiTpyy+/VHx8vFasWGF5Fz3ZzZs3lS9fvjQfB0DucuLECY0bN04ODg767rvvUhXZkkeDPq7ly5crX7582rVrl1xdXVO0RUVFWf7v5+en3bt368KFCxo5cqTKlSuXoq/ZbNaIESN048YNLVq0yFLwlO6PovHx8dGIESN0/PhxFSxYMMWxW7Zs0fLly9WtWzfLuXx9ffXDDz+offv2mjRpkkaNGmXpP2bMGC1fvlwrVqxIceP6xhtv6PDhw3rrrbdSXCPi4+PVv39/ffHFF+ratas6d+6c4vF/++039e7dW0FBQY9UNIyNjdU333yjOnXq6Icffkh1zD+/bomJiXrhhRcUGxursLAwNW/e3NJ25coVtW3bVhMmTFCHDh1UokSJFOfZvn273n//fQ0cODDF/n79+umHH37QypUrU/08rF27VnFxcerTp4/s7B784TNrrs3vvfeeNm3apKFDh2r27Nmyt7eXdP910bhx47R8+XItXbpUI0aMeOBjWvu4LVu21B9//GF5wzLZkSNH1LlzZ7399ttas2bNQx8zLY/6sw8gd+L+M3vff2bm427evFlLlixR7969Lfs++eQTjRs3TkuWLNG777772Pkf9vpLuj969syZM1q6dKl8fX0t+2/cuKEuXbpo7ty58vb2tjyf5cuXq1SpUtq3b58KFSqU4lxc/7IHphdApvv8888l3b/I/XOhiHz58lmKXNYOlU9r3hY7OzvLjd+2bdseN24qGzZsUHR0tHr06JHigidJr7/+epqLX6xatUp///23Bg8enOKCJ92/oSlVqpS2bNmiS5cupTp2xIgRlguPdP+dso4dO8psNqtTp06WC490/2vYvXt3SdIvv/xi9XO7du2a1qxZo+LFi2vu3Lmpbk4dHBxSjFpN5uvrm2qUTvKN508//ZRif+nSpVMVXCWpXbt2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D7m5uVk1D6k11+YJEybot99+02effaYtW7aoZcuWKlOmjCIjI/Xnn39q//79GjJkSJoLljzu49rZ2Wn48OF677335Onpqc6dO+vevXvatWuXoqOj1aJFi3RHVj8Ka372AeRO3H9m3/vPrKpTp06qWrWqQkNDdfHiRTVq1EhXrlzRxo0b1aFDhzSnxnvQ6y9JWrp0qbp166YxY8ZoyZIleuaZZ+Ts7KxLly7pxIkTOnbsmD7//HOVLVtWd+/eVffu3eXm5qZnnnlGZcuW1b1797R7924dP35cDRs2THdRVGQdFF2RIezs7LRy5UpNmzZNW7Zs0Y0bN2Q2m1W6dGk9/fTTWrJkiTw9PbVixQqtWLFCSUlJqlSpkvz9/TVixAjlzZs3xflmz54tOzs7bd++XQcOHFBiYqKef/55derUSaVKldLGjRv11ltvaf/+/dq2bZs8PDwUGBgoLy8vm1/0JKlbt25as2aN5syZo2+//Vb58uWTp6entm7dqvfeey/NuUIHDRqkihUrauHChVq/fr1u376tUqVKadiwYRo/fvxjLQ6SEZycnLR582YtWbJEa9as0cqVKyXd/0hm165d1bRp0yc6/8KFC1W+fHmFhoZq6dKlKl68uDp16qQ33njD8i60NUwmkz7//HMFBgbqq6++0ocffihXV1f16dNHgwcP1oYNG9Kcq2ju3Llq3769li1bpt27dys6OlpFihRR6dKlNXjw4BQrdg8ZMkTOzs46fPiwDh48qHv37qlUqVLq06ePRo8e/cRFAQA5R//+/VWjRg0tXLhQe/fu1ZYtW+Ts7CwPDw+9+uqrKfr27t1bZrNZCxYs0JdffqmnnnpKbdq00YoVKzRjxoxU537rrbe0bds2HT9+XD/88IPy5MkjNzc3TZw4UcOHD08xj+x///tfXbx4UatWrVJQUJDu3bunsmXLWl70169fXzt27NC8efO0Y8cO/fDDDypSpIi8vb312muvPVJB7kk4Ojrqu+++04oVK/T111/ru+++U1xcnFxcXOTu7q6pU6fapJDm7u6uN954Q7t27dKePXsUGRmpIkWKqGLFihozZoz69u2bon+NGjW0Z88eBQcHa8OGDfriiy9kNpvl6uqqatWq6eWXX5aHh8djZenXr5+2bNmie/fuqU+fPqneEHwQa67NefLk0WeffWbpt3XrVsvCkGXLltW4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gRR9TCYTH38EAMBKNWrU0K5du9K8uTSbzQoLC2MRLQBAjjJq1Kh024oXL67x48dr/PjxmZgIAADbsaroWrx4cbm4uMjDwyOj8gAAkCuNHDlSQ4YM0dy5c9WjRw9JUlJSkv744w/NmjVLP//8s7766iuDUwIAYDtHjx5Ntc9kMsnZ2VlPPfWUAYkAALAdq4qu69evz6gcAADkaj179tSFCxc0Y8YMzZ4927JPkuzt7TV9+nS1a9fOyIgAANiUu7u70REAAMgwVhVdAQBAxhk7dqx8fX21bt06nTlzRklJSapQoYKeffZZlStXzuh4AABkmNu3bys6OlpmszlVW9myZQ1IBADAk3msouu9e/cUHh6u2NhYJSUlpWpv1qzZEwcDACA3cnNze+AcdwAA5BR///235s6dq+XLlysqKirdftevX8/EVAAA2IZVRVez2awZM2ZoyZIlun37drr9uCgCOYfd3o+MjgDkCvv379e+ffs0bty4NNvfe+89NWvWTI0aNcrkZAAAZIzXX39dn332mTp16qRmzZrJycnJ6EgAANiMVUXXBQsWKDAwUP3791ezZs00YsQIvf322ypSpIg+/PBD5cmTR/7+/hmVFYABkjyHGh0BORyF/fvmzJnzwJvNX375Rbt379aaNWsyLxQAABnom2++Ud++ffXBBx8YHQUAAJuzqui6YsUKdenSRQsWLLCMZq1Tp468vLzUp08ftW3bVrt375aXl1eGhAUAIKc6duyYxo8fn277M888o4CAgExMBABAxkpKSlLDhg2NjgEgC2FABnISq4quFy5csMwzZ2dnJ+n+PDySlD9/fvXu3VtLlizR5MmTbRwTAICc7c6dOzKZTA/sc+vWrUxKAwBAxmvbtq3279+vgQMHGh0FQBbBJy2RGTKruG9nTWcnJyfduXNHklS4cGHly5dPFy9etLTnz5+f+VwBAHgMlStX1tatW9Nt37JliypWrJiJiQAAyFhz587VL7/8opkzZyoiIsLoOAAA2JRVI12rV6+uY8eOSbo/0rV+/fpaunSp2rVrp6SkJH366afy8PDIkKAAAORkAwYM0Ouvv65XX31VkydPVrFixSRJUVFRmjlzpnbs2KEZM2YYnBIAANupVauWzGazAgICFBAQoLx581o+UZnMZDLp0qVLBiXMXKNffkXXoyKNjpHt9O3b1+gI2UrRYsX1wcIFRscAcgWriq69evXS0qVLFRcXJwcHB02dOlU9evRQrVq1JEl58+ZVSEhIhgQFACAnGzp0qI4fP65PPvlEn376qVxcXGQymXT16lWZzWb17dtXI0eONDomAAA24+Pj89CpdXKT61GRGjmdRcWQsYKnjDY6ApBrWFV07devn/r162fZbtq0qfbt26eNGzfK3t5ebdu2VaVKlWweEgCA3GDBggXq1auX1q1bp7Nnz8psNqtChQrq1q2bmjdvbnQ8AABsKjg42OgIAABkGKuKrmkpX758jhx5w0c7Hg8f7bAeH+8A8E8tWrRQixYtjI4BAAAAAHgCj110vX37tqKjo2U2m1O1lS1b9olCZQV8tAOZhY93AAAAIDfYs2ePJKlZs2Ypth8muT8AANmJVUXXv//+W3PnztXy5csVFRWVbr/r168/cTAAAHKyLl26yM7OTqGhocqTJ4+6du360GNMJpPWrVuXCekAALC9Ll26yGQy6cqVK8qXL59lOz1ms1kmk4n7SwBAtmRV0fX111/XZ599pk6dOqlZs2ZycnLKoFgAAORsZrNZSUlJlu2kpKSHLiaS1qdLAADILsLCwiRJ+fLlS7ENAEBOZFXR9ZtvvlHfvn31wQd87B4AgCexfv36B24DAJDT/HtRSBaJBADkZHbWdE5KSlLDhg0zKgsAAAAAAAAAZHtWjXRt27at9u/fr4EDB2ZQHAAAcqdatWqpZ8+e8vHxUZ06dYyOAwBAprh27ZpWrVqls2fPKiYmJtVUOiaTSR999JFB6QAAeHxWFV3nzp0rHx8fzZw5U4MHD5arq2tG5QIAIFepUaOGFi1apAULFqhixYrq2bOnevTooapVqxodDRls0KBBio+Pl4ODgz7++GOj4wBAptmwYYMGDx6suLg42dvbq1ChQqn6PGy+cwAAsqoHFl1LliyZ6iKXmJio3377TQEBAcqbN6/s7FLOUGAymXTp0qVHevArV67orbfe0tatW3Xr1i2VL19egYGBlrl9zGazZs+ereXLlysmJkYNGjRQQECAqlevbs1zBAAgy/vqq68UExOjdevWae3atQoMDNS8efNUvXp1+fr6ysfHR+XLlzc6JjJAfHy8JCkuLs7gJACQuSZPnqzSpUvro48+Ur169SiwAgBylAcWXX18fDLswhcTE6MOHTqoSZMmWrVqlYoVK6Zz587JxcXF0mf+/PkKCgpSUFCQPDw8LCNtDx06JEdHxwzJBQCAUZycnDRgwAANGDBAkZGRWrt2rUJDQ/XOO+/onXfeUf369fX9998bHRM2NGjQoBTbL774IqNdAeQaEREReuutt1S/fn2jowAAYHMPLLoGBwdn2AMvWLBAJUuW1JIlSyz7/jmCx2w2Kzg4WGPHjlW3bt0seTw8PLR69epUNykAAOQkxYsX19ChQzVo0CCFhIRoypQp+umnn4yOBRtLHuWajNGuAHKTOnXqKDIy0ugYAABkCLuHd8kY69evV4MGDTRo0CBVrlxZzZs314cffmiZOP3cuXOKiIhQmzZtLMcUKFBAnp6eOnDggFGxAQDIFLt27dK4ceNUrVo1jR07Vnnz5mUhSwBAjjJ9+nR99tln2rVrl9FRAACwOasW0nrvvfe0adMmbd68Oc32jh07ytvbWy+//PJDz3X27FktW7ZMo0aN0tixY3X8+HFNnDhRkjRs2DBFRERIUorpBpK3L1++/MBzh4eHP8rTAbIMfmaR29nqd8DDw8Mm5zHKwYMHtWbNGq1bt04RERF66qmn1LlzZ/n6+qp169ayt7c3OiIAADbToEEDzZw5U927d5ebm5vKlCmT6lpnMpm0bt26RzqfLdYMiY+P15QpU7RmzRrFxcWpZcuWCgwMVJkyZWz3xAEAuYJVRdevv/46xcjTf2vUqJG+/PLLRyq6JiUlqV69epo2bZqk+x8tOXPmjJYuXaphw4ZZ+v17Tlmz2fzQeWaz+003ch9+ZpHb8Tsg1apVSxcvXpSDg4Pat2+vHj16qEOHDsqfP7/R0ZCB8ufPn2KKAQcHBwPTAEDm+vrrrzVy5EiZzWbFx8c/0VQDtlozxM/PTxs2bNCyZcvk7OysyZMnq3fv3tq5cydvfgIArGJV0fXs2bMPvDGuVKnSIy/+4OrqqqpVq6bYV6VKFf3111+Wdkm6evWq3NzcLH0iIyNTjX4FACC7q1Gjht588015e3urUKFCRsdBJvnkk0/Ut29fyzaLaAHITaZPn67q1atr5cqVcnd3f6Jz2WLNkNjYWK1YsUJBQUFq3bq1JGnJkiWqVauWduzYobZt2z5RRgBA7mLVnK758+d/4Ef7L126JDu7RztlkyZNdOrUqRT7Tp06pbJly0qSypUrJ1dXV23fvt3SHhcXp3379qlx48bWxAYAIEuLi4tT/fr1Vbx4cQquuVDyaGZGuQLIba5du6aBAwc+ccFVss2aIUeOHNG9e/dS9HFzc1PVqlVZVwQAYDWrRro2atRIK1as0PDhw+Xs7JyiLTo6WitXrnzkguioUaPUvn17BQQEqEePHjp27Jg+/PBDvfnmm5LuTyswcuRIBQYGysPDQ5UrV1ZAQIAKFSokX19fa2IDAJClOTg46L333tPcuXONjgIDfPLJJ0ZHAABDNGzYUOfOnbPJuWyxZsjVq1dlb2+vYsWKpepz9erVdB+b9RmQ3fAzC9jm9+Bh0+RZVXSdNGmSOnXqpGbNmmnkyJGqUaOGTCaTfv31Vy1evFiRkZH69NNPH+lc9evX18qVK+Xv76958+bJzc1Nb7zxhoYMGWLpM2bMGN29e1cTJkywTHQeGhpqmW8HAICc4umnn9aZM2eMjgEAQKaZN2+eevXqpVq1aqlXr15PdK6MXDPkYX2Ymx7ZDT+zQOb8HlhVdK1Xr56++uorjRkzRlOnTrVceMxms8qXL6+vvvpKDRs2fOTzdejQQR06dEi33WQyyc/PT35+ftbEBAAg25k2bZpeeOEFNW3a9IHXRgAAcooBAwbo77//1vDhwzV27FiVKlUq1WJVJpNJ+/fvf+i5bLFmSIkSJZSYmKioqCgVL148RR9PT8/He5IAgFzLqqKrJHl5eennn3/W0aNH9eeff8psNqtixYqqU6fOQ98hBAAAaVuwYIGcnJz0/PPPq3Tp0ipfvrwKFCiQoo/JZNKqVasMSggAgG0VL15cLi4uqly58hOfy5o1Q+rXry/p/9YM8ff3lyTVrVtXefPm1fbt2y0jby9evKiTJ0+yrggAwGpWF12l+zd9devWVd26dR/YLyoqSm3atNFHH32kRo0aPc5DAQCQK5w4cUImk8ky+ub8+fOp+vDmJgAgJ1m/fr3NzmWLNUOKFCmi/v37a+rUqXJxcZGzs7MmT56smjVrqlWrVjbLCgDIHR6r6PqoEhMTdf78ed29ezcjHwYAgGzv+PHjRkcAACBLe9CgHlutGTJz5kzZ29tr0KBBiouLU8uWLbV48eJU0x4AAPAwGVp0BQAAAADAFh42qMcWa4Y4ODho3rx5mjdv3hPnBQDkbnZGBwAAAPclJiZq1apVGj16tHr37q1ffvlFkhQTE6O1a9fqypUrBicEAAAAADwKiq4AAGQBsbGxat++vYYPH65vv/1WW7duVVRUlCTJ0dFRkydP1ocffmhwSgAAAADAo6DoCgBAFvD222/rxIkT+vrrr3XkyBGZzWZLm729vbp27aqtW7camBAAAAAA8KgougIAkAWsX79ew4YN03/+8x+ZTKZU7ZUqVdKFCxcMSAYAAAAAsBZFVwAAsoCYmBhVqFAh3Xaz2ay///47ExMBAAAAAB5XhhZd7e3tVbZsWRUoUCAjHwYAgGzP3d1dv/32W7rte/bsUeXKlTMxEQAAAADgceXJyJMXK1ZMx44dy8iHAAAgR+jVq5fef/99de3aVdWrV5ckyzQDS5Ys0XfffaeZM2caGREAAEPlhkE9wVNGGx0BAGAjVhddjx8/rs8//1xnz55VTExMioU+pPs3iJs3b7ZZQAAAcoNx48bpf//7n5599llVrlxZJpNJkyZN0vXr1xURESFvb28NHz7c6JgAABgmNwzqGTn9A6MjIIejsA9kHquKrp9++qleffVV2dnZqUyZMipcuHBG5coS+GMEAMgsefPm1apVq/T111/rm2++kclkUkJCgurUqaMePXroueeeS3OBLQAAsjMG9QAAciqriq5z585V3bp1FRISopIlS2ZUpiyDdxmRGSjuA/inXr16qVevXkbHAAAgw+W2QT0AgNzFqqLrjRs3NH78+FxRcAUAwGjx8fEKCwtTTEyMOnbsKDc3N6MjAQBgM7ltUA8AIHexs6ZzkyZNdPr06YzKAgBArjV+/Hg1b97csp2QkKAOHTpo2LBhmjBhgpo2bapff/3VwIQAANjWjRs39N///peCKwAgR7Kq6DpnzhyFhYUpJCREiYmJGZUJAIBcZ+fOnerQoYNle+3atTp69KgCAgK0detWFStWTPPmzTMwIQAAtsWgHgBATmbV9AKVKlXS+PHj9fLLL2vs2LEqUaKE7O3tU/QxmUw6cuSILTMCAJDjXb58WeXKlbNsb9iwQU8//bRefPFFSdKLL76oxYsXGxUPAACbmzNnjnx8fFSzZk317t071b0lAADZmVVF16CgIL355pt66qmnVK1aNSY6BwDARvLkyaO7d+9Kksxms3788Uf179/f0u7k5KTr168bFQ8AAJtjUA8AICezqui6cOFCNWvWTF9++aUKFSqUUZkAAMh1atSooVWrVql3794KCwtTdHS0/vOf/1jaz58/r+LFixuYEAAA22JQ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\n",
      "text/plain": [
       "<Figure size 1512x504 with 3 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Generate boxplots for total_dom_charges, customer service calls and total_dom_minutes.\n",
    "plt.figure(figsize=(20,15), dpi=300)\n",
    "boxplot(\"churn\", [\"total_dom_charges\", \"customer service calls\", \"total_dom_minutes\"], df)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "e10140eb",
   "metadata": {},
   "source": [
    "Plot insights:\n",
    "* Churning customers have total domestic charge median of $64.\n",
    "* Churning customers have a median customer service call rate of 2 calls.\n",
    "* Churning customers have a median domestic minute usage of 650 minutes."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "733e314a",
   "metadata": {},
   "source": [
    "### Feature selection using ANOVA"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "336b18a2",
   "metadata": {},
   "source": [
    "#### ANOVA is used for comparing the distribution of a numeric variable in two or more groups\n",
    "* Ho = Null Hypothesis = the distribution of the varible in multiple groups is uniform\n",
    "* Ha = Alternate Hypothesis = the distribution of the variable in multiple groups in different\n",
    "    \n",
    "    we analyse the pvalue, lets say for confidence interval of 95%, significance level = 5%\n",
    "\n",
    "`if pvalue>0.05 = accept the null hypothesis and the feature is NOT important`\n",
    "`if pvalue <0.05 = reject the null hypothesis and the feature is important`"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "9e083890",
   "metadata": {},
   "source": [
    "#### ANOVA summary:\n",
    "\n",
    "ANOVA stands for Analysis of Variance and it is a statistical method for testing if the means of different groups are equal. In Python, ANOVA can be performed using the f_oneway function from the scipy.stats library. This function takes in multiple sets of data, each representing a group, and returns a test statistic and p-value. The test statistic measures the difference between the means of the groups and the p-value gives the probability of observing a difference as large or larger than what was observed if the means were actually equal. If the p-value is low, we can reject the null hypothesis that the means are equal, implying that there is a significant difference between the means of at least two groups.\n",
    "\n",
    "#### In simple terms:\n",
    "\n",
    "ANOVA, or Analysis of Variance, is a statistical method for comparing multiple groups or populations to see if their means are significantly different from each other. In simple terms, it tells us if the difference between the groups' averages is due to chance or if it's real. To perform ANOVA in Python, we can use the f_oneway function from the scipy.stats library. This function takes in multiple arrays of data, and returns a test statistic and a p-value. The p-value represents the probability of observing the difference between the groups if they are actually all the same. If the p-value is below a certain threshold (usually 0.05), we reject the null hypothesis (that the groups have the same mean) and conclude that at least one of the groups is different."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 34,
   "id": "4af3a557",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "Index(['state', 'account length', 'area code', 'phone number',\n",
       "       'international plan', 'voice mail plan', 'number vmail messages',\n",
       "       'total day minutes', 'total day calls', 'total day charge',\n",
       "       'total eve minutes', 'total eve calls', 'total eve charge',\n",
       "       'total night minutes', 'total night calls', 'total night charge',\n",
       "       'total intl minutes', 'total intl calls', 'total intl charge',\n",
       "       'customer service calls', 'churn', 'state_churn_rate',\n",
       "       'total_dom_minutes', 'total_dom_charges', 'vmail_messages'],\n",
       "      dtype='object')"
      ]
     },
     "execution_count": 34,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.columns"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 35,
   "id": "8ffd4266",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "account length 0.33976000705720666\n",
      "number vmail messages 2.1175218402696038e-07\n",
      "total day minutes 5.300278227509361e-33\n",
      "total day calls 0.28670102402211844\n",
      "total day charge 5.30060595239102e-33\n",
      "total eve minutes 8.011338561256927e-08\n",
      "total eve calls 0.5941305829720491\n",
      "total eve charge 8.036524227754477e-08\n",
      "total night minutes 0.04046648463758881\n",
      "total night calls 0.7230277872081609\n",
      "total night charge 0.040451218769160205\n",
      "total intl minutes 8.05731126549437e-05\n",
      "total intl calls 0.002274701409850077\n",
      "total intl charge 8.018753583047257e-05\n",
      "customer service calls 3.900360240185746e-34\n"
     ]
    }
   ],
   "source": [
    "numerics =['account length','number vmail messages',\n",
    "       'total day minutes', 'total day calls', 'total day charge',\n",
    "       'total eve minutes', 'total eve calls', 'total eve charge',\n",
    "       'total night minutes', 'total night calls', 'total night charge',\n",
    "       'total intl minutes', 'total intl calls', 'total intl charge',\n",
    "       'customer service calls']\n",
    "xnum = df[numerics]\n",
    "y = df['churn']\n",
    "from sklearn.feature_selection import f_classif\n",
    "fval,pval = f_classif(xnum,y)\n",
    "for i in range(len(numerics)):print(numerics[i],pval[i])"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "99abe8cc",
   "metadata": {},
   "source": [
    "### Feature selection using Chi Square Test\n",
    "\n",
    "- Used to compare the distribution of categories of a categorical feature in two or more groups\n",
    "- in nutshell to compare whether a categorical attribute has some relationship with the other categorical attribute\n",
    "\n",
    "* H0 = Null Hypothesis = the categorical attribute has uniform distribution in two or more groups\n",
    "* Ha = Alternate hypothesis = the categorical attribute has different distribution in two or more groups\n",
    "\n",
    "We always analyse the pvalue, consider 95% as confidence interval, significance level = 5% i.e.0.05\n",
    "\n",
    "`if pvalue >0.05 = accept the Null hypothesis - feature is not important`\n",
    "`if pvalue <0.05 = reject the Null hypothesis - feature is important`"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "68470e82",
   "metadata": {},
   "source": [
    "#### Chi Square test summary:\n",
    "The chi-square test is a statistical test used to determine if there is a significant association between two categorical variables. It measures the difference between the expected frequencies of the variables and the observed frequencies in a sample data. If the difference is too large, it suggests that the variables are not independent and there is a relationship between them. The test returns a p-value, which represents the probability of observing the difference by chance if the variables are independent. A small p-value indicates that it is unlikely that the relationship between the variables is due to chance and suggests that the variables are associated.\n",
    "\n",
    "#### In simple terms:\n",
    "Chi-Square test is a statistical test used to determine if there is a significant association between two categorical variables.\n",
    "\n",
    "Imagine you have a table with two categories, \"red\" and \"blue\". The Chi-Square test can tell you if there is a relationship between the two categories. For example, if you have a table that shows the number of red and blue candies in a jar, the Chi-Square test can tell you if there is a significant difference in the number of red and blue candies in the jar.\n",
    "\n",
    "It's like asking if the red and blue candies are evenly distributed in the jar or if one color is more common than the other. The test helps you answer this question by comparing the expected number of red and blue candies with the actual number you observe. If the difference is too big, it suggests that there is a relationship between the two categories.\n",
    "\n",
    "In simple terms, the Chi-Square test is a way to see if there is a connection between two things by counting how many of each thing there are and comparing the result with what you expect to see."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 36,
   "id": "d566dbdf",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "state [0.19214979]\n",
      "area code [0.89394206]\n",
      "phone number [1.91173945e-14]\n",
      "international plan [4.09173473e-46]\n",
      "voice mail plan [5.28486023e-07]\n",
      "vmail_messages [0.0396314]\n"
     ]
    }
   ],
   "source": [
    "categories = ['state','area code','phone number', 'international plan',\n",
    "       'voice mail plan','vmail_messages']\n",
    "\n",
    "y = df['churn']\n",
    "from sklearn.preprocessing import LabelEncoder\n",
    "from sklearn.feature_selection import chi2\n",
    "for col in categories:\n",
    "    xcat = LabelEncoder().fit_transform(df[col]).reshape(-1,1)\n",
    "    cval,pval = chi2(xcat,y)\n",
    "    print(col,pval)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 37,
   "id": "6ae5343d",
   "metadata": {},
   "outputs": [],
   "source": [
    "#selecting important features based on previous analysis\n",
    "x = df[['international plan','vmail_messages','total day minutes','total eve minutes',\n",
    "     'total night minutes','total intl minutes','customer service calls']]\n",
    "y = df['churn']"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "7221b527",
   "metadata": {},
   "source": [
    "# 5. Preprocessing"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 38,
   "id": "f3374dba",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>international plan</th>\n",
       "      <th>vmail_messages</th>\n",
       "      <th>total day minutes</th>\n",
       "      <th>total eve minutes</th>\n",
       "      <th>total night minutes</th>\n",
       "      <th>total intl minutes</th>\n",
       "      <th>customer service calls</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>no</td>\n",
       "      <td>Normal Users</td>\n",
       "      <td>265.1</td>\n",
       "      <td>197.4</td>\n",
       "      <td>244.7</td>\n",
       "      <td>10.0</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>no</td>\n",
       "      <td>Normal Users</td>\n",
       "      <td>161.6</td>\n",
       "      <td>195.5</td>\n",
       "      <td>254.4</td>\n",
       "      <td>13.7</td>\n",
       "      <td>1</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>no</td>\n",
       "      <td>No VM plan</td>\n",
       "      <td>243.4</td>\n",
       "      <td>121.2</td>\n",
       "      <td>162.6</td>\n",
       "      <td>12.2</td>\n",
       "      <td>0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>yes</td>\n",
       "      <td>No VM plan</td>\n",
       "      <td>299.4</td>\n",
       "      <td>61.9</td>\n",
       "      <td>196.9</td>\n",
       "      <td>6.6</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>yes</td>\n",
       "      <td>No VM plan</td>\n",
       "      <td>166.7</td>\n",
       "      <td>148.3</td>\n",
       "      <td>186.9</td>\n",
       "      <td>10.1</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "  international plan vmail_messages  total day minutes  total eve minutes  \\\n",
       "0                 no   Normal Users              265.1              197.4   \n",
       "1                 no   Normal Users              161.6              195.5   \n",
       "2                 no     No VM plan              243.4              121.2   \n",
       "3                yes     No VM plan              299.4               61.9   \n",
       "4                yes     No VM plan              166.7              148.3   \n",
       "\n",
       "   total night minutes  total intl minutes  customer service calls  \n",
       "0                244.7                10.0                       1  \n",
       "1                254.4                13.7                       1  \n",
       "2                162.6                12.2                       0  \n",
       "3                196.9                 6.6                       2  \n",
       "4                186.9                10.1                       3  "
      ]
     },
     "execution_count": 38,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "x.head()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "717f17fb",
   "metadata": {},
   "source": [
    "#### encoding categorical features"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 39,
   "id": "ca2d7fc0",
   "metadata": {},
   "outputs": [],
   "source": [
    "from sklearn.compose import ColumnTransformer\n",
    "from sklearn.preprocessing import OneHotEncoder,OrdinalEncoder,StandardScaler\n",
    "preprocessor = ColumnTransformer([('ohe',OneHotEncoder(),[1]),\n",
    "                                ('ode',OrdinalEncoder(),[0]),\n",
    "                                 ('sc',StandardScaler(),[2,3,4,5,6])],remainder='passthrough')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 40,
   "id": "4185f37c",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>0</th>\n",
       "      <th>1</th>\n",
       "      <th>2</th>\n",
       "      <th>3</th>\n",
       "      <th>4</th>\n",
       "      <th>5</th>\n",
       "      <th>6</th>\n",
       "      <th>7</th>\n",
       "      <th>8</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.566767</td>\n",
       "      <td>-0.070610</td>\n",
       "      <td>0.866743</td>\n",
       "      <td>-0.085008</td>\n",
       "      <td>-0.427932</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>-0.333738</td>\n",
       "      <td>-0.108080</td>\n",
       "      <td>1.058571</td>\n",
       "      <td>1.240482</td>\n",
       "      <td>-0.427932</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.168304</td>\n",
       "      <td>-1.573383</td>\n",
       "      <td>-0.756869</td>\n",
       "      <td>0.703121</td>\n",
       "      <td>-1.188218</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>2.196596</td>\n",
       "      <td>-2.742865</td>\n",
       "      <td>-0.078551</td>\n",
       "      <td>-1.303026</td>\n",
       "      <td>0.332354</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>-0.240090</td>\n",
       "      <td>-1.038932</td>\n",
       "      <td>-0.276311</td>\n",
       "      <td>-0.049184</td>\n",
       "      <td>1.092641</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "     0    1    2    3         4         5         6         7         8\n",
       "0  0.0  0.0  1.0  0.0  1.566767 -0.070610  0.866743 -0.085008 -0.427932\n",
       "1  0.0  0.0  1.0  0.0 -0.333738 -0.108080  1.058571  1.240482 -0.427932\n",
       "2  0.0  1.0  0.0  0.0  1.168304 -1.573383 -0.756869  0.703121 -1.188218\n",
       "3  0.0  1.0  0.0  1.0  2.196596 -2.742865 -0.078551 -1.303026  0.332354\n",
       "4  0.0  1.0  0.0  1.0 -0.240090 -1.038932 -0.276311 -0.049184  1.092641"
      ]
     },
     "execution_count": 40,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "x_new = preprocessor.fit_transform(x)\n",
    "pd.DataFrame(x_new).head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 70,
   "id": "c51ffb8e",
   "metadata": {},
   "outputs": [],
   "source": [
    "# train Test Split before preprocessing"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "b1da6c93",
   "metadata": {},
   "source": [
    "#### Train Test Split"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 41,
   "id": "71957e10",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "(3333, 7)\n",
      "(2666, 9)\n",
      "(667, 9)\n",
      "(3333,)\n",
      "(2666,)\n",
      "(667,)\n"
     ]
    }
   ],
   "source": [
    "# train test split\n",
    "from sklearn.model_selection import train_test_split\n",
    "xtrain,xtest,ytrain,ytest = train_test_split(x_new,y,test_size=0.2,random_state=5)\n",
    "print(x.shape)\n",
    "print(xtrain.shape)\n",
    "print(xtest.shape)\n",
    "print(y.shape)\n",
    "print(ytrain.shape)\n",
    "print(ytest.shape)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "0d485bcc",
   "metadata": {},
   "source": [
    "# 6. Apply Machine Learning algorithm - Logistic regression"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a3fad7e5",
   "metadata": {},
   "source": [
    "#### Logistic Regression Summary:\n",
    "\n",
    "Logistic Regression is a machine learning technique used for predicting a binary outcome (yes/no, 0/1) based on a set of independent variables. In Python, it can be implemented using the LogisticRegression module from the scikit-learn library. The algorithm creates a mathematical model to estimate the probability of the binary outcome based on the input variables, and then predicts the output class based on a threshold value (e.g., 0.5). The parameters of the model are learned from the training data through an optimization process. Logistic Regression can be used for various types of problems, including binary classification and multinomial classification.\n",
    "\n",
    "#### In simple terms:\n",
    "\n",
    "Logistic Regression is a machine learning algorithm used for classification problems. It is used to predict a binary outcome (1 or 0) based on one or more independent variables. In simpler terms, it helps us predict whether an event will occur or not.\n",
    "\n",
    "Imagine you have data about people and their income, education level, and age, and you want to predict whether a person is likely to buy a car or not. Logistic Regression can help us make that prediction based on the information we have about those people.\n",
    "\n",
    "In Python, you can implement Logistic Regression using the scikit-learn library. First, you will split your data into a training and test set. Then, you will use the training set to fit a Logistic Regression model. Finally, you will evaluate the model using the test set to see how accurate it is in making predictions."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 42,
   "id": "6c7acc9c",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "LogisticRegression(class_weight='balanced')"
      ]
     },
     "execution_count": 42,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "from sklearn.linear_model import LogisticRegression\n",
    "model = LogisticRegression(class_weight='balanced')\n",
    "model.fit(xtrain,ytrain)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "936ae660",
   "metadata": {},
   "source": [
    "# 7. Performance Analysis\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 43,
   "id": "b07479a1",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.775112443778111\n",
      "Recall :  0.7934782608695652\n",
      "F1 score :  0.4932432432432432\n",
      "Precision :  0.35784313725490197\n"
     ]
    }
   ],
   "source": [
    "# performance analysis\n",
    "from sklearn import metrics\n",
    "ypred = model.predict(xtest)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytest,ypred))\n",
    "print(\"Recall : \",metrics.recall_score(ytest,ypred))\n",
    "print(\"F1 score : \",metrics.f1_score(ytest,ypred))\n",
    "print(\"Precision : \",metrics.precision_score(ytest,ypred))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "0fc204a1",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.775 means 77.5% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.793 means 79.3% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.49 means the balance between precision and recall was 49%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.35 means 35.7% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 44,
   "id": "6c2fed17",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.764066016504126\n",
      "Recall :  0.7570332480818415\n",
      "F1 score :  0.4848484848484849\n",
      "Precision :  0.3566265060240964\n"
     ]
    }
   ],
   "source": [
    "# performance analysis on train data\n",
    "ypred2 = model.predict(xtrain)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytrain,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytrain,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytrain,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytrain,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "0074344e",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.76 means 76% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.76 means 75.7% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.48 means the balance between precision and recall was 48%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.35 means 35.6% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "e0dbac1a",
   "metadata": {},
   "source": [
    "### Observation - Recall is not satisfactory\n",
    "So we will now try other algorithms"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "55a13b12",
   "metadata": {},
   "source": [
    "# Improving and Fine-Tuning"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "38f3a9d1",
   "metadata": {},
   "source": [
    "1. Improving Data:\n",
    "* Enhance feature engineering\n",
    "* Obtain additional data - features or samples\n",
    "* Enhance data preprocessing steps\n",
    "* Refine feature selection process - eliminate irrelevant features\n",
    "2. Improving Modeling:\n",
    "* Adjust hyperparameters to optimize algorithm performance\n",
    "* Utilize a different machine learning algorithm\n",
    "* Implement ensemble methods combining multiple algorithms for improved predictions."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "96b83067",
   "metadata": {},
   "source": [
    "## Decision Tree Classifier"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 45,
   "id": "986bd728",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>0</th>\n",
       "      <th>1</th>\n",
       "      <th>2</th>\n",
       "      <th>3</th>\n",
       "      <th>4</th>\n",
       "      <th>5</th>\n",
       "      <th>6</th>\n",
       "      <th>7</th>\n",
       "      <th>8</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>265.1</td>\n",
       "      <td>197.4</td>\n",
       "      <td>244.7</td>\n",
       "      <td>10.0</td>\n",
       "      <td>1.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>161.6</td>\n",
       "      <td>195.5</td>\n",
       "      <td>254.4</td>\n",
       "      <td>13.7</td>\n",
       "      <td>1.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>243.4</td>\n",
       "      <td>121.2</td>\n",
       "      <td>162.6</td>\n",
       "      <td>12.2</td>\n",
       "      <td>0.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>299.4</td>\n",
       "      <td>61.9</td>\n",
       "      <td>196.9</td>\n",
       "      <td>6.6</td>\n",
       "      <td>2.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>166.7</td>\n",
       "      <td>148.3</td>\n",
       "      <td>186.9</td>\n",
       "      <td>10.1</td>\n",
       "      <td>3.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3328</th>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>156.2</td>\n",
       "      <td>215.5</td>\n",
       "      <td>279.1</td>\n",
       "      <td>9.9</td>\n",
       "      <td>2.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3329</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>231.1</td>\n",
       "      <td>153.4</td>\n",
       "      <td>191.3</td>\n",
       "      <td>9.6</td>\n",
       "      <td>3.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3330</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>180.8</td>\n",
       "      <td>288.8</td>\n",
       "      <td>191.9</td>\n",
       "      <td>14.1</td>\n",
       "      <td>2.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3331</th>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>213.8</td>\n",
       "      <td>159.6</td>\n",
       "      <td>139.2</td>\n",
       "      <td>5.0</td>\n",
       "      <td>2.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3332</th>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>1.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>234.4</td>\n",
       "      <td>265.9</td>\n",
       "      <td>241.4</td>\n",
       "      <td>13.7</td>\n",
       "      <td>0.0</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>3333 rows ?? 9 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "        0    1    2    3      4      5      6     7    8\n",
       "0     0.0  0.0  1.0  0.0  265.1  197.4  244.7  10.0  1.0\n",
       "1     0.0  0.0  1.0  0.0  161.6  195.5  254.4  13.7  1.0\n",
       "2     0.0  1.0  0.0  0.0  243.4  121.2  162.6  12.2  0.0\n",
       "3     0.0  1.0  0.0  1.0  299.4   61.9  196.9   6.6  2.0\n",
       "4     0.0  1.0  0.0  1.0  166.7  148.3  186.9  10.1  3.0\n",
       "...   ...  ...  ...  ...    ...    ...    ...   ...  ...\n",
       "3328  0.0  0.0  1.0  0.0  156.2  215.5  279.1   9.9  2.0\n",
       "3329  0.0  1.0  0.0  0.0  231.1  153.4  191.3   9.6  3.0\n",
       "3330  0.0  1.0  0.0  0.0  180.8  288.8  191.9  14.1  2.0\n",
       "3331  0.0  1.0  0.0  1.0  213.8  159.6  139.2   5.0  2.0\n",
       "3332  0.0  0.0  1.0  0.0  234.4  265.9  241.4  13.7  0.0\n",
       "\n",
       "[3333 rows x 9 columns]"
      ]
     },
     "execution_count": 45,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# preprocessing pipeline\n",
    "from sklearn.compose import ColumnTransformer\n",
    "from sklearn.preprocessing import OneHotEncoder,OrdinalEncoder,StandardScaler\n",
    "preprocessor = ColumnTransformer([('ohe',OneHotEncoder(),[1]),\n",
    "                                ('ode',OrdinalEncoder(),[0])],\n",
    "                                remainder=\"passthrough\")\n",
    "x_new = preprocessor.fit_transform(x)\n",
    "pd.DataFrame(x_new)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "24445e7a",
   "metadata": {},
   "source": [
    "# train test split\n",
    "xtrain, xtest, ytrain, ytest = train_test_split(x_new, y, test_size=0.2, random_state=5)\n",
    "# don't need to do train test split again\n",
    "# Print shapes of data\n",
    "print(\"x shape:\", x.shape)\n",
    "print(\"xtrain shape:\", xtrain.shape)\n",
    "print(\"xtest shape:\", xtest.shape)\n",
    "print(\"y shape:\", y.shape)\n",
    "print(\"ytrain shape:\", ytrain.shape)\n",
    "print(\"ytest shape:\", ytest.shape)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 47,
   "id": "d45bf19f",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "DecisionTreeClassifier(class_weight={0: 0.5, 1: 0.5}, random_state=5)"
      ]
     },
     "execution_count": 47,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# decision tree\n",
    "from sklearn.tree import DecisionTreeClassifier\n",
    "model2 = DecisionTreeClassifier(random_state=5,class_weight={0:0.5,1:0.5})\n",
    "model2.fit(xtrain,ytrain)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "121522ee",
   "metadata": {},
   "source": [
    "### Visualizing the tree model"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 48,
   "id": "30befe46",
   "metadata": {
    "scrolled": false
   },
   "outputs": [
    {
     "data": {
      "image/png": "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qLLn66+/xvjx45OWjR8/Hl999VVW2m9ubkZxcfKsuTNnzsTq1auz0v5gsSwLp9OJoqIjCeJZs2bh3XffzWFUA7Nx40ZMnTo18feoUaOwd+/eHEaUe8FgEIsXL4ZIJML999/f70Rep9LSUixduhSfffYZXnvttcRs0fmotrYW5eXJvfDOP/98rFmzJkcRDUwkEkEoFIJSeaQUwhVXXIG33norh1FlBiXzSF7w+XxYsmQJmpqa8Pjjj6O0dPDDY88991zce++9eO655wrmBIAQQoaKrVu34tFHH8Vtt92Wll7Sc+bMwY033ogFCxZg+/btaYiQdHR04E9/+hOWLVuGu+66CzfffHPa77rLZDL8+te/xm233Ya//e1veP755xEIBNLaBkn2zTff4NRTk2ekFAqFEIvF8Hq9OYqqcLz77rvdjlkmkwk2my2vL8DT5cMPP8TFF1+ctOyiiy7CRx99lJX2V65ciblzkyepOOmkk7Bz586stD9YX3zxBc46K7kupUwmQywWQyjU95DMXHM4HNDpdODxktMEVVVV2LdvX46iyq29e/fikUcewU033dRt3xgIPp+PO+64A8OGDcO8efPgdrvTGGX6rFq1CldccUXSsokTJ+LLL7/MUUQD88knn+D8889PWqbRaODxeBCLxXIUVWZQMo/k3KZNm/Doo4/ipptuwtVXX53WLvxqtRrz5s2DRCLBvHnz0N7enrbXJoQQ0l04HMZTTz2F3bt3Y+nSpWm5OdPJbDZj6dKl+Pbbb/H0008jEomk7bWPJ+FwGP/85z/xzDPP4LrrrsNdd92VdAc7E/R6PR544AFcfPHFWLJkCd54440hd1KdL/73v/+lvOCcPXs23n777RxEVFh2796dchjc6aefjm3btuUgouzx+XwQCATdeh2JRCIIhcKMJ4M5joPdbkdJSfcaXJWVlTh06FBG20+HdevWYfr06d2WX3LJJfjwww+zH9AArVy5EnPmzOm2/Hg9fixfvhzvv/8+lixZktTbeTCmTp2KO++8E4899hh27NiRltdMl1gsBq/X220kBcMwKC4uRnNzc24CG4Avv/wyZUmJ6dOn4/PPP89BRJlDyTySM36/H0uXLkVdXR2WLFkCo7H3ae4H4/zzz8fdd9+Nv/71r/jggw8y1g4hhBzPdu3ahYceegiXX345brjhhozUV+LxeLj55ptxySWX4MEHHzzuh/4MBMdxePvtt7Fw4UKcddZZeOSRR9KabO2PYcOGYdGiRRg5ciQefPBBfPbZZ8dFb6ds8fv94PP5EIvF3R4bOXLkcduzpr8OHDiA4cOHp3zsoosuGvJDlVP1Sux0+eWX45133slo+1u2bOmxrmMhDJNzuVxQKpXg87tPFHHqqafim2++yUFU/cdxHKxWa8qklVKpRCgUOm5uonm9XixYsAAGgwF33303BIL0TjWg1+uxZMkSfPnll3jppZfy5nfw888/x4wZqWejnTt3bt6XG0g1TL/TWWedhY0bN+YgqsyhZB7Jic66PD/72c9w7bXXZuSC72gajQbz58+HQCDAvHnzqHYPIYSkCcuy+Pvf/47PPvsMS5Ys6fFiOJ1GjBiBxx9/HB999BH+9a9/gWXZjLdZyDZu3Ij7778fJSUleOyxxzBq1KicxjN+/HgsXboUkUgEDz74IL799tucxjNUvPfee70Oa6+ursaePXuyGFFheeuttzB79uyUj2Wrd1ou9dQrEQBGjx6d8Zsnn376KS644IKUj2k0Gni93rzu0ZtqeGInhmFgMpnQ2NiY5aj6L1W9xK4KqW7aYOzcuROLFi3Cr371qx4TW+nA4/Hwy1/+EqeccgoefPDBvLg2PbpeYlclJSWw2+15k3hMZeXKlbjyyitTPsbj8aDT6eBwOFI+XogomUeyKhAI4IknnsDBgwexdOlSmM3mrMdw4YUX4u6778Zf/vKXgujuTggh+ay+vh733XcfJk6ciF//+tdpv3vdG6FQiN/85jc49dRT8cADD6CpqSlrbReKXbt24cEHH0RbWxuWLl2KyZMn5zqkBIZhcNFFF+HRRx/Fd999h/nz56Ouri7XYRW0Xbt29TpT4uzZszPeu6pQRaNR+P1+qNXqHtfJRu+0XNm/fz+qqqp6Xae6ujpjvTs9Hg8kEkmvvyHTp0/H2rVrM9J+OtTX1/c6W+bcuXOxatWq7AU0QB999FGvNeEKqW7aseA4Dq+++irWrVuHJUuWpBzunQmnn3467rvvPjz55JM5/Xx7qpfY1aRJk7Bly5YsRtV/HMehtbW11+021CYzomQeyZqtW7diwYIFuO666/DTn/40K73xeqLRaLBw4UJwHIf58+fD5XLlLBZCCClEHMfhzTffxJtvvolHH30Up5xySs5iOfXUU7FgwQK8+uqreX2hlE1WqxWPPvooNm/enLYZajNFIBDgZz/7Ge677z689957eOKJJ/Kih0KhOXjwIIYNG9brOkqlEuFw+LgZKjcQn376Kc4777xe18lG77Rcefvtt3vsldgpk3XT+tN+Pg+T++6773DiiSf2uk5RURGcTmde9iTvqV5iV4VUN22gOjo68Mgjj6C6uhq//vWvUw6VziSNRoPHHnsMu3btwvPPP5+T70hP9RK7Ov/88/Hpp59mKaKB2bx5MyZNmtTrOmazGVarNa97Fw4EJfNIxgUCAfzhD3/A3r17sXTp0rQVD02HSy65BHfeeSeeeuqprM3SRQghhc7hcOChhx6C2WzG73//e0gkklyHBKlUigceeABFRUV45JFHjttkkNvtxlNPPYVly5bhzjvvxC233JL2GWozRSaT4Y477sCtt96KF154Ac8//zyCwWCuwyoYvQ0R7eqCCy4Y8rXfjkVNTU2/eq5msndaroRCIQQCAahUql7XUyqVCAaDGUkGHzhwANXV1b2uk8/D5N5//31ceumlfa531llnYd26dZkPaIB6q5fYVSHUTRuob775BkuWLMFdd92V097rDMPgpptuwtSpU/HAAw/AbrdnrW2O42Cz2focNScQCCCRSODxeLIUWf/1Nky/q/Hjx2P79u1ZiCjzKJlHMmrr1q2YP38+rr32Wlx//fV52StAp9Nh0aJFiEQiWLBgwXF7AUgIIX1hWRb/+9//8Nxzz+G+++7DWWedleuQupk+fTruuece/OUvf8nbu8eZEA6H8dJLL+GZZ57Btddei7vuuqvPC/N8pdfr8eCDD+Kiiy7C448/jmXLluVlT5Z80tbWBqvV2usQ0U4TJkzAU089lYWoCofD4cDu3bv7dZ46e/ZsvPDCC1mIKnv+85//9Ou7A8R75qxevTqt7X/44Ydoa2vr17pz5szBM888k9b2B6utrQ0sy0Iqlfa57oQJE/DXv/41C1ENzD//+U+ccMIJfa5XUlKCvXv3DonakYFAAH//+9/x1VdfYcmSJSgqKsp1SACAk08+GQ8//DCeffZZrFmzJiu9yN59912MHTu2X+vOnj0bb775ZoYjGpj9+/cjGo32q9TLxRdfjHfffTcLUWUeJfNIxrzwwgtYsWIFnnjiCVgsllyH06eZM2fijjvuwOzZs2n4CSGEpPD4449j9+7dmD9/fr8v/HKh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1157LWvWrOEHP/hB1OX3Vd577z2ys7M555y2hrwmpk6dyhdffMGnn37K2LFj2y0nEESDYDDIF198QUFBAQ6HA5VKxejRo7nxxhux2+29rV67SJLE0KFDGTp0KACyLPPtt9+yevVqKitDXrEjR45k2rRpJCZGPxPrmYDT6cRisbTyIJ43bx5vvPEGN998c9TlB4NBnE5ns1cgwKxZs3jiiSfOWmPeqaxfv5677rqr+e/ExERqamqQZbldrxGBQBB7/vvf/7Jr1y4eeughtNqub4dRqVTcfvvtvP322zzyyCPcddddaDT9blrU73E4HCxfvpzFixeHvCQ7QJIk7rrrruZ3PXr06NgoKRAIBH2Uftdrff3119xwww2tjl1//fX8+9//5je/+U3U5b/33nvcc889rY5ddtllPPzwwzEx5q1cuZLvf//7rY6NGDGC//znP1GX3VcpKipi586dLFy4sNOyN998MwsWLCA3NzcqHjACQROyLPP1119TUFBAVVUVkiQxcuRIfvjDH/bruqdSqRgxYgQjRowAQoahL7/8kv/85z/U1NSgUqkYNWoUU6dO7dNGyt5k1apVbTxBsrKyKCkpQYlBhuHNmzdz4YUXtjpmNBpRFAWPx9MqJuPZiNvtRpKkNs9h+vTpbNmypc2zEwgEsUeWZZ555hlSU1O59957u32fq666iry8PO69917uuecesSgVQ+rq6njwwQdZtGgRNputS9dIksS9997LAw88gE6nax6LCAQCwdlIvzLmFRUVhQ3InpSUFJMV8/r6erRabZuVP61Wi16vp66u7rS2x50uiqJQVVVFcpgtBIMGDeLgwYMMHhyliOl9FI/HwzPPPMOyE1k5O0OSJO655x6WLVvGww8/LDwsBBFDURT27dvH5s2bOX78OBAytM+dO5eUlJRe1i56qNVqRo0axahRo4DQ9uE9e/bwr3/9C6fTiVqtZuzYsUyePLnLg/UznaKiouZtzC0ZM2YMn332WdQ9hzdt2sTixYvbHJ85cybr169n3rx5UZXf13nrrbeYOXNmm+MzZszggQceEMY8gaCXqaurY/ny5dxwww0R8c4aOnQo9913Hw899BDf//73m/szQfRobGzk/vvv55577jnthT9Jkli0aBFLlizhxhtvZMgQkZlYIBCcnfQrY97KlSv5xS9+Efbc1KlTKSwsZPr06VGT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9rOw5eKAdS+/THajngsuuCC2SkaYzz77jNmzZzf/PWHCBHbs2MHEiRN7TymBQCDoAhG3br399ttceeWVrY7l5uZSVFQUaVFhWbFiReuJM3DBBRewZcuWmMgXdIzT6cRkMqHRnLQjjx8/np07d8ZEvizLOByOVl5N6enpHD9+vFU8s1izcuVKzj///LCeO01oNBp+//vf88gjj/SqroL2kWWZNWvWcN9995GZmcmyZcs4//zzIyojOTmZu+++mx/96Ec899xzPPXUUzgcjojK6O+88847XHHFFa2OzZ07l9WrV8dE/ueff87Ika3j7syePZs333wzJvIFAoEgFjQ0NHD//aEFWnsHhrwmLBYL9913H8uWLcPpdEZfwT7A6tWrWxmKAM455xy+/vrr3lGoC1RWVrJ+/Xp++tOfdljuRz/6ER9++CGlpaWxUSwKFBcXk5mZ2coL7/LLL+e///1vL2olEAgEXSOixrzGxkbUajV6fevsTnPnzmXlypWRFBUWRVGoqKggLa21h8RFF13Exo0boy5f0Dlvvvkmc+fObXVMkiTS0tJiMhjYvHlz2BXE8ePHs2PHjqjLD8e3337L0aNHueyyyzotm5KSwpVXXsmLL74YA80EXUVRFNatW8eiRYtISUlh2bJlTJgwofMLe0Bqair33HMP3//+93n22Wf54x//eNZMjjrj008/Zdy4ca2O5eXlcfDgwZjIf+utt5g5c2arYzabDbfbTSDQd5MUCQQCQVfxeDzk5+dz9913n1bYh7i4OBYuXMgDDzxAfX19FDXsfXw+H36/v81uilmzZrFmzZpe0qpjAoEAf/jDH5g/f36n20wlSeLuu+/mySefxO/vnwm7Vq1axXXXtU7CodVq0el01NXV9ZJWAoFA0DUiasxbu3Yts2bNanPcbrdTX19PMBiMpLg27NixI+wEWq1WY7PZqK0VGYl6m8OHDzNo0KA2x6+77rqYGHw3bdrEhRde2Ob4FVdcwXvvvRd1+adSV1fHCy+8wG9+85suXzNlyhR8Pl/MvBkF7aMoCm+//XazV8Ly5cuZPHlyTHVISwttv77++ut55plnePrpp8/qAWhpaSnp6elhJyHDhg3jm2++iap8j8eDoiitY1ue4NJLL2XDhg1RlS8QCATRxufzsXjxYn7729+SknL68a/j4+OZP38+S5cuxe2OTnbxvsC7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P2358uVZPdeOb3leVVVFQ0ODXG+FEEKIDJNknhBiStm1axeLFy8eNG358uVs3749K/U//fTTXHfddYOmeTweGhsbp8WXm5dffplLLrlk0DRFUSgpKaG5uTnj9e/bt485c+YMmX7OOeewa9eujNcv4MMPP+S0004bNO2SSy7hlVdeGaeI0uONN97g9ttvZ8+ePdx222384z/+Iw6HI+P1LlmyhHvuuYdFixZx//338/DDD9Pd3Z3xesXE9/zzz3P55ZcPmqbT6cjPz6et7eQe9T4R7733HqeffvqQ6QsWLOCdd97JeP1CCCHEdCbJPCHElNHc3ExxcfGQR/kURaGgoIDW1taM1h8Oh1FVFZttaGff0+XLzfbt21m+fPmQ6dlqnbhx40auv/76IdOvuOIKXnjhhYzXP90dOnSIGTNmDJluMBiw2Wx0dXWNQ1QnT9M0XnzxRW677TaOHj3K3XffzRe+8AXM5vQPojOS0047jTvvvJMrrriC//iP/+Chhx6ivX1of4Vi+njrrbc4++yzh0xfs2YNTzzxRMbr37RpU9LuE66++mo2b96c8fqFEEKI6UySeUKIKWP9+vWsXbs26bzVq1dnPJn07LPPcvXVVyedd9VVV035Lzetra3k5+cn7RetsLCQtra2jLZOjEajhMNhnE7nkHlGoxGj0SgtmjKspqaGVatWJZ13ww03sGHDhixHdHJisRg1NTV873vfw2AwcO+997Jy5coJ0W9ddXU1t99+O3/7t3/Lr3/9a+6//36OHDky3mGJLGtqaqKkpCTp9basrIzm5uaMXm9DoRCqqmK1Dh0Z3GKxoCgKwWAwY/ULIYQQ050k84QQU4KmaXi93v4R/Y5XVlaW8X7T9uzZw4IFC5LOM5vN6HS6Kf3lZv369axZsybl/Ez3m/bCCy8MeeRsoFWrVk2aZNJklKy/xIFmzpzJoUOHshzViQmFQjz66KPcddddVFVVce+993LRRRdNyIE7ioqK+M53vsNXv/pVnnrqKe6++24+/vjj8Q5LZElNTc2w19tFixZltK/SZ599lmuuuSbl/GuvvZZNmzZlrH4hhBBiupNknhBiSti+fTvLli0btkwm+007fPgw5eXlw37pv+6666bslxtN02hpaaGsrCxlmUz3m/bmm28O6S9xoLlz57J///6M1T/dvfLKK1x88cXDlpk3bx4ffPBB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eFwSeVOALqcYfdmp/X/xgtn87qXdGD3z0ZediqH8NDZs/5CArWxQOX3ZqZLIG6WWHpW7f76eqiVXYvCcgb5iftI/c/VCCuZfwG0/+R0h90xJ5E0xOrsbXX71oL+f/XEztop5/a+NRbP45ZMvDimny6+WRJ4QQgiRRZLME0JMK5s3b+aaa64ZNG3VqlVs3LhxfAKaZPx+PxaLBYPh014azjzzTN57772s1K+qKp2dneTnf5okLCgooKOjA1VVsxKDGF/PPfccV1555aBpq1ev5sknnxyniCa31tZWHnroIe66665B/U+mkpeXx7/8y79w5513EgwGsxChGC/xeJyenh5cLlf/tJKSEo4ePSo/lAghhBDjTJJ5Qohp5b333uPMM88cNG327NkcPHhwnCKaXDZu3MiqVauGTK+srKSuri7j9W/bto0VK1YMmb5ixQpef/31jNcvxt/bb7/NWWedNWhadXU19fX14xTR5NXR0cEPf/hD7rzzTiwWy6iXKyoq4lvf+hbr1q0jEkk9aJCY3F555RUuumhov32LFy9m586d4xCREEIIIfpIMk8IMW3U19cnHRgBYO7cuXz00UdZjmjyOXjwILNnzx4yPVsto7Zs2ZL0y+XFF1/Mq6++mvH6xfhqbGyktLQ0aZ9up59+Ou++++44RDU5+Xw+7rvvPu644w7sdvsJL19WVsbXv/511q1bRzQqI9xORal+PLn88sv585//PA4RCSGEEKKPJPOEENPGk08+yZo1a5LOW7lypTxqO4KPPvqIU045Jek8l8tFIBAgHo9nrH6fz4fD4UCv1w+Zp9frcTgc+Hy+jNUvxl9NTQ1r165NOu+aa65h8+bNWY5ocgoEAtxzzz1873vfIycn56Tfp7KykhtvvJG77roro+e+yL62tjby8vKGDEAEYDQaMZvNBAKBcYhMCCGEECDJPCHENBGPx/H7/YP6/hnIbrcTi8XkkbFhbNy4keuvvz7l/AsvvDCjreOefPJJVq9enXL+6tWrqampyVj9YnypqkpHR8eg/hIHslqtaJpGKBTKcmSTS29vL3fddRff/e53U14PT8TMmTP5u7/7O37wgx9Iv5VTyPr161P++AXS16wQQggx3iSZJ4SYFrZu3cqFF144bJkrrriC559/PjsBTTKRSIRYLDbs43jnnXce27Zty1gM9fX1VFdXp5wv/aZN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0d3djdFoxGhMjKSqKArl5eU0NDSMc2RCTC319fV897vfZdWqVSeUyAO4/vrr+au/+ituu+026urqMhPgBNfQ0MDOnTuTJvIgce269dZbeeCBB1BVNcvRDbZx40auv/76/tcrV67kqaeeGr+ARL8///nPXH755f2v16xZQ01NzThGJIQQQojRkGSeEEIMcPyXTkh8uXnyySfHJyAhpqDNmzfzu9/9jnvvvZdZs2ad1HvMmDGDe++9lz/+8Y/TLjEUDof5z//8T26++eZhy7lcLv72b/+Wn/3sZ1mKbKhIJEI0GsXh+LQ7ArvdTiwWIxJJ3SebyLzu7m7MZnP/j1cA+fn5dHZ2jnsCWAghhBDDk2SeEEIMsH//fubOnTtoWl5eHl1dXfLlRogxCofD3HfffQDcdtttmM3mMb2f2Wzm1ltvxWQycc899xAKhdIR5oT3wx/+kG984xuj2n5nnHEG+fn5vPrqq5kPLInnn3+eK664Ysj0K664gueee24cIhJ9kv14BXDeeefx+uuvZz8gIYQQQoyaJPOEEOKY/fv3p2wldN555/Haa69lOSIhpo6DBw9y++2384UvfKG/f650ufLKK/nSl77EHXfcwf79+9P63hPNE088wdKlS6msrBz1Mp///Od57bXXaGxszGBkye3evZuzzz57yPRFixbx9ttvZz0e8an9+/dzyimnDJl+4YUXjlvyVwghhBCjI8k8IYQ4ZsOGDaxatSrpvAsvvJCtW7dmOSIhpoaamho2bNjA/ffff0JJqBNRUVHB/fffz6ZNm3j88cczUsd4++CDD2hqauLSSy894WVvvvlm/v3f/z2rj7Y2NTVRUlKCoihD5imKQklJCU1NTVmLR3xq3759zJ49O+k8vV6P0+nE5/NlNyghhBBCjJok84QQgsSomuFwGKfTmXS+TqcjJyeHzs7OLEcmxOTV29vL3XffTW5uLt/5zncG9c2VCQaDgX/+53+msLCQO++8k+7u7ozWl01+v59HH32Ur3/96ye1vNVq5Wtf+xo/+tGP0hxZauvXr085QAfA2rVrWb9+fdbiEZ/auHFjyh+vAG644QYZCEMIIYSYwCSZJ4QQJEb0u+yyy4YtI19uhBi9Dz/8kHXr1nHjjTeeVEuysbjooov4yle+wt13383777+f1bozQdM0HnzwQW655Rb0ev1Jv091dTULFy5kw4YNaYwuOVVVaWtro7CwMGWZgoIC2tvbpT/SLItGo0QikUGDkhyvurpaRnEXQgghJjBJ5gkhBLBz506WLFkybJmqqir5ciPECDRN43e/+x0vvfQSDzzwAKWlpeMSR3FxMffffz9btmzhscceQ9O0cYkjHX75y19yww03kJ+fP+b3uuKKKzh06BD79u1LQ2Sp1dbWsnz58hHLLV++nG3btmU0FjHYCy+8wGc/+9kRy51++um8++67WYhICCGEECdKknlCiGmvpaWFoqKipP06He+MM85g7969mQ9KiEnI7/fz/e9/n8rKSm666aYxtSJLB71ez0033cTs2bO544476OrqGtd4TsYzzzyD0Whk0aJFaXvPb3zjG/ziF7+grq4ube95vFdeeYVLLrlkxHKXXHIJW7ZsyVgcYqhdu3axePHiEctdffXVfPe7381CREIIIYQ4UZLME0JMe1/72tdG1UoB4OKLL5YvN0IkUVNTw4033sg//dM/cd555413OIMsW7aMb33rW3zlK1/hD3/4w3iHc0Ief/xx/v7v/z6t72kwGFizZg2///3v0/q+fY4ePcquXbswGAwjltXr9bz//vv85S9/yUgsYrD333+fffv2jerHK5vNxv3335+FqIQQQghxohSfzzd5nzsRQog0aG9vT8vja0JMZ52dnZhMJux2+3iHklIwGKS3t1fO9wmmq6sLm82W8QFShBBCCCGmCknmCSGEEEIIIYQQQggxSYz8/IMQQgjUrma0gHdUZRVnIbrc8en0X4ixUrtaCHc0YzIZUVUVVVX7H5eMRKKYTKlbTymOAnS5JdkK9aSo3e1EurxJ12Ok9cPqQucYW6u+vmtJJBrFZEyyjY9NTyYd1xa1u42I7yTXH8DmQucokGviBHay+0ZRFGKxGCaTaUi5SCSSdPqgelV1Ug80I4QQQkwmkswTQkwLzd0q3mDiS8ZzT/6BMk81doeTcChINBplweJl/WULrQqljsFdimoBL6GaWwB4fG8rVW4LDrOeUFQlGtdYWp3TX9ay+kGQL65iktq25UWitb/G47bQ6AvhybOy57CfAruRHKsRfyhGocNEMBrHbNDhD8YAKHKamPXFf4UJnsyrffVl4u9tojLfSUN7AACdogxKQswucRGNq+h1CgePJgbNsBj0LP7SHTDGZJ4W8LLlR18FwOM20+gL43Fb2HMk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jl26dCmtViuNRqPbUQAAgHWgr9sBAOBOUxRFKpVbf5NFp9NJWV7/t7VarZZarbZS0QAAgB6nzAOAFVapVDIwMHDL8y9evJjl5eVVTAQAANwplHkAsMKOHz+e/v7+TExMZGZmJhcuXMjQ0FCazWbq9Xr6+/tTlmUWFhZSFEX27t17zT06P55LOTedJHnt33+fiR1jadx9dxZarSwtLeU3f3//lblFYySVTaNrtTwAAKCLvDMPAFZYtVrNwMBA2u12FhcXMzk5mZGRkWzfvv1Kgffggw9emX+9J/OWv//vLE4euaXP23Do5VR3/N2KrgEAAFifPJkHAKukr68vAwMDee655277Hh+dvpDp+aUMb7wrSbJ764a0O2Vmfmrnb7dtTFEUKxUXAADoAco8AFhFx48fz9TUVEZGRpIk99xzT9rtdmZmZnLffff97LX/8ccfs3PzhhRFsrFWzZaNfTnTbKXV7qRWreTk6bmMb6ln11osBAAAWBeUeQCwCt5///1MTEykKIrs2bMnW7duzTfffJMvv/wy9Xo9RVHk7Nmz2b59+w3v8Y8Tm5IkOzbXkyQ/XLyUfTsaV81pLrRXbxEAAMC6o8wDgFXwyCOPJEnGx8eTJLOzs9m/f39qtdqVObOzsze8/sOPT2Xpjxeyqb+askzOz13KUH9f5lvLOT93KUlS76tkW6N2w3sAAAB3HhtgAMAKK4oiJ06cSJIMDQ3l7NmzKcsyw8PDWVhYSKfTSbVaTb1ez+joaMbGxlKWV38d///dbG/6eXazBQCAXwxlHgCsovn5+bzyyis5cuTyzrSff/55Pvvsszz77LNdTgYAAPSiSrcDAMCd7M0338xTTz115Xjv3r354osvupgIAADoZco8AFhFX331Ve69996rzu3atSvffvttlxIBAAC9TJkHAKvk66+/zu7du685//TTT2dycrILiQAAgF6nzAOAVfLGG2/k4MGD15zftGlT5ufn02631z4UAADQ05R5ALAK2u12FhYWMjg4eN3xxx9/PO+8884apwIAAHqdMg8AVsHbb7+dJ5544objDz/8cE6cOLGGiQAAgDuBMg8AVsHJkyezf//+G44XRZEtW7Zkenp6DVMBAAC9TpkHACtsamoqIyMjKYriZ+c988wzOXr06BqlAgAA7gTKPABYYa+//noOHz5803ljY2M5f/58yrJcg1QAAMCdQJkHACuoLMscPXo027Ztu+X5r7766iqnAgAA7hRFs9n0OAAArKC5ubk0Go1uxwAAAO5AnswDgBWmyAMAAFZLX7cDAMAvUefHcynnbm0n26Ixksqm0VVOBAAA9AJlHgDchnMXO5leuPymit//22sZ2zmRuwcaaS0uZGlpKff/+jdJkpH+IqMD1z4IX85NZ3HySJLkX0/9kPGhDRmoV7O41MnScpn9E4NX5m449HKizAMAAKLMA4DbMr1Q5sixxcsHW3+bLOTy31+89pexl/9pQ0YHbnyfj05fyN21asoymVtczu6tG9LulPl6eiF7RvpXbwEAAEBPUuYBwF/hwh8+ytKP07mrMZwk2fCr3Sk77XQW59M/uuem1/96fPCmcwAAAP6XMg8AbsOpkx+m9adtSYps2DaRvoEtaf1wOq3p00lRJEWR1uy5JLuue/2HH5/KtmYr5y60srFWzZaNfTk900qZMrVqJUvLZca31DM6WF/TdQEAAOubMg8AbsP9Dz6U+rHF1LfuSJK055sZvGf/VXPa880bXv/QP9yfxe/q2bH5clnXXGhn346B1Pr+7/16zYX2ygcHAAB6mjIPAG7DSH+R39U/SYoijU2bk7JMa/H7pChyqdXKH774LIf/+V8y0l9c9/oTn3+bjP8umwcbKcvk3A/TGdo8mKFNgzk39ackSb1ey/LIcHY0RtZwZQAAwHpWNJvNstshAAAAAICbq9x8CgAAAACwHijzAAAAAKBHKPMAAAAAoEco8wAAAACgRyjzAAAAAKBHKPMAAAAAoEco8wAAAACgRyjzAAAAAKBHKPMAAAAAoEco8wAAAACgRyjzAAAAAKBHKPMAAAAAoEco8wAAAACgR/wPxIHdT6m8fXUAAAAASUVORK5CYII=\n",
      "text/plain": [
       "<Figure size 1440x720 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "import matplotlib.pyplot as plt\n",
    "from sklearn import tree\n",
    "\n",
    "fname = ['International plan', 'vmail_NO_Plan','vmail_Normal','vmail_HF', 'Total day minutes',\n",
    "       'Total eve minutes', 'Total night minutes', 'Total intl minutes',\n",
    "       'Customer service calls']\n",
    "cname = ['Not Leaving','Leaving']\n",
    "\n",
    "plt.figure(figsize=(20,10))\n",
    "tree.plot_tree(model2, feature_names=fname, class_names=cname, filled=True)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 49,
   "id": "d4a47db2",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.8995502248875562\n",
      "Recall :  0.7391304347826086\n",
      "F1 score :  0.6699507389162561\n",
      "Precision :  0.6126126126126126\n"
     ]
    }
   ],
   "source": [
    "# performance analysis\n",
    "ypred2 = model2.predict(xtest)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytest,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytest,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytest,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytest,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "b6f21733",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.89 means 73% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.739 means 73.9% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.669 means the balance between precision and recall was 66.9%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.612 means 61.2% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 50,
   "id": "e2d5274f",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  1.0\n",
      "Recall :  1.0\n",
      "F1 score :  1.0\n",
      "Precision :  1.0\n"
     ]
    }
   ],
   "source": [
    "\n",
    "# performance analysis on train data\n",
    "ypred2 = model2.predict(xtrain)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytrain,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytrain,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytrain,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytrain,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "de5b5d08",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 1 means 100% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 1 means 100% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 1 means the balance between precision and recall was 100%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 1 means 100% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "352a8047",
   "metadata": {},
   "source": [
    "## Overfitting\n",
    "Test data performance: low\n",
    "\n",
    "Train data performance: high\n",
    "\n",
    "Causes of Overfitting\n",
    "\n",
    "Complex model that is too specific to the training data\n",
    "Too many features or not enough data to support the complexity of the model\n",
    "Using features that are not relevant to the target\n",
    "Solutions for Overfitting\n",
    "\n",
    "Reduce the complexity of the model\n",
    "Use regularization techniques like L1 or L2 regularization\n",
    "Use early stopping to stop training when performance on validation set starts to decrease\n",
    "Use cross-validation to get a better estimate of model performance\n",
    "Remove irrelevant features from the data set\n",
    "Increase the size of the training data set."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "cc6600ab",
   "metadata": {},
   "source": [
    "## Underfitting\n",
    "`performance of model on test data = low`\n",
    "\n",
    "`performance of model on train data = low`\n",
    "\n",
    "\n",
    "**Reasons for underfitting**\n",
    "- lack of informative features\n",
    "- lack of a powerful algorithm, as the existing features may have silghtly complex/nonlinear relation with the target and the current algorithm is not able to learn\n",
    "- presence of noisy observations\n",
    "\n",
    "\n",
    "**Ways to handle underfitting situation**\n",
    "- colllect/ create more features, perform feature extraction\n",
    "- collect more columns, NO BENFIT from collecting rows\n",
    "- Try a more powerful/complex predictive algorithm\n",
    "- In case of deicision tree, increase the value of max_depth, decrease the value of min_samples_leaf and min_samples_split\n",
    "- perform better data cleaning, handling outliers etc."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d15a3b2d",
   "metadata": {},
   "source": [
    "## Optimal Model\n",
    "Test Data Performance = High\n",
    "\n",
    "Train Data Performance = High"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a5d5bb16",
   "metadata": {},
   "source": [
    "## Hyperparameter Tuning for decision tree using Gridsearch"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 51,
   "id": "238521b6",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Fitting 5 folds for each of 5808 candidates, totalling 29040 fits\n"
     ]
    },
    {
     "data": {
      "text/plain": [
       "GridSearchCV(cv=5, estimator=DecisionTreeClassifier(random_state=5), n_jobs=-1,\n",
       "             param_grid={'max_depth': array([ 3,  5,  7,  9, 11, 13, 15, 17, 19, 21, 23]),\n",
       "                         'min_samples_leaf': array([ 3,  5,  7,  9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31, 33, 35,\n",
       "       37, 39, 41, 43, 45, 47, 49]),\n",
       "                         'min_samples_split': array([ 10,  15,  20,  25,  30,  35,  40,  45,  50,  55,  60,  65,  70,\n",
       "        75,  80,  85,  90,  95, 100, 105, 110, 115])},\n",
       "             scoring='recall', verbose=True)"
      ]
     },
     "execution_count": 51,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "param_grid = {\"max_depth\":np.arange(3,25,2),\n",
    "              \"min_samples_leaf\":np.arange(3,50,2),\n",
    "              \"min_samples_split\":np.arange(10,120,5)}\n",
    "from sklearn.model_selection import GridSearchCV\n",
    "grid_search = GridSearchCV(DecisionTreeClassifier(random_state=5),\n",
    "                          param_grid=param_grid,n_jobs=-1,\n",
    "                          scoring='recall',verbose=True,cv=5)\n",
    "grid_search.fit(x_new,y)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 52,
   "id": "de47c969",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "0.6708762886597939\n",
      "{'max_depth': 15, 'min_samples_leaf': 3, 'min_samples_split': 10}\n"
     ]
    }
   ],
   "source": [
    "print(grid_search.best_score_)\n",
    "print(grid_search.best_params_)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 53,
   "id": "8049a051",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "DecisionTreeClassifier(max_depth=8, min_samples_leaf=5, min_samples_split=20,\n",
       "                       random_state=5)"
      ]
     },
     "execution_count": 53,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Controlling overfitting\n",
    "model2 = DecisionTreeClassifier(criterion='gini',random_state=5,\n",
    "                               max_depth=8,min_samples_leaf=5,min_samples_split=20)\n",
    "model2.fit(xtrain,ytrain)\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 54,
   "id": "be57cfc9",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.9430284857571214\n",
      "Recall :  0.7065217391304348\n",
      "F1 score :  0.7738095238095237\n",
      "Precision :  0.8552631578947368\n"
     ]
    }
   ],
   "source": [
    "# performance analysis On test data\n",
    "ypred2 = model2.predict(xtest)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytest,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytest,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytest,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytest,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "7eac49be",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.943 means 94.3% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.70 means 70% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.77 means the balance between precision and recall was 77%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.85 means 85% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 55,
   "id": "333d74d0",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.9471117779444861\n",
      "Recall :  0.6956521739130435\n",
      "F1 score :  0.7941605839416058\n",
      "Precision :  0.9251700680272109\n"
     ]
    }
   ],
   "source": [
    "# performance analysis on train data\n",
    "ypred2 = model2.predict(xtrain)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytrain,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytrain,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytrain,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytrain,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "b070d215",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.947 means 94.7% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.695 means 69.5% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.794 means the balance between precision and recall was 79.4%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.925 means 92.5% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "2e98dc1a",
   "metadata": {},
   "source": [
    "### Feature importances"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 56,
   "id": "edb38d5c",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "International plan 0.0\n",
      "vmail_NO_Plan 0.07957197040961295\n",
      "vmail_Normal 0.0\n",
      "vmail_HF 0.1034922637532645\n",
      "Total day minutes 0.35860654052548413\n",
      "Total eve minutes 0.1721940195076953\n",
      "Total night minutes 0.04746707503384585\n",
      "Total intl minutes 0.08934222663734767\n",
      "Customer service calls 0.1493259041327496\n"
     ]
    }
   ],
   "source": [
    "model2.feature_importances_\n",
    "for i in range(len(fname)):print(fname[i],model2.feature_importances_[i])"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "4babfeff",
   "metadata": {},
   "source": [
    "## Random Forest"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 57,
   "id": "f747be91",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "RandomForestClassifier(max_depth=8, oob_score=True, random_state=5)"
      ]
     },
     "execution_count": 57,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "import random\n",
    "from sklearn.ensemble import RandomForestClassifier\n",
    "\n",
    "random_state = 5\n",
    "n_estimators = 100\n",
    "max_depth = 8\n",
    "oob_score = True\n",
    "\n",
    "model4 = RandomForestClassifier(n_estimators=n_estimators, random_state=random_state,\n",
    "                                max_depth=max_depth, oob_score=oob_score)\n",
    "\n",
    "model4.fit(xtrain, ytrain)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 58,
   "id": "02549722",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.9445277361319341\n",
      "Recall :  0.6521739130434783\n",
      "F1 score :  0.7643312101910829\n",
      "Precision :  0.9230769230769231\n"
     ]
    }
   ],
   "source": [
    "# performance analysis On test data\n",
    "ypred2 = model4.predict(xtest)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytest,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytest,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytest,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytest,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "7445cd16",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.944 means 94.4% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.652 means 76.4% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.764 means the balance between precision and recall was 76.4%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.923 means 92.3% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 59,
   "id": "de807146",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.959489872468117\n",
      "Recall :  0.7237851662404092\n",
      "F1 score :  0.8397626112759643\n",
      "Precision :  1.0\n"
     ]
    }
   ],
   "source": [
    "# performance analysis on train data\n",
    "ypred2 = model4.predict(xtrain)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytrain,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytrain,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytrain,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytrain,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "572154fc",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.959 means 95.9% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.723 means 72.3% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.84 means the balance between precision and recall was 84%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 1 means 100% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 60,
   "id": "0ef1f5cc",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0.9358589647411854"
      ]
     },
     "execution_count": 60,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "#check OOB (out of bag) score\n",
    "model4.oob_score_"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a739e968",
   "metadata": {},
   "source": [
    "## Adaboost"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 61,
   "id": "37fda2d6",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "AdaBoostClassifier(learning_rate=0.2, n_estimators=120, random_state=5)"
      ]
     },
     "execution_count": 61,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "from sklearn.ensemble import AdaBoostClassifier\n",
    "model5 = AdaBoostClassifier(n_estimators=120,random_state=5,learning_rate=0.2)\n",
    "model5.fit(xtrain,ytrain)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 62,
   "id": "d155800e",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.8830584707646177\n",
      "Recall :  0.32608695652173914\n",
      "F1 score :  0.43478260869565216\n",
      "Precision :  0.6521739130434783\n"
     ]
    }
   ],
   "source": [
    "# performance analysis On test data\n",
    "ypred2 = model5.predict(xtest)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytest,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytest,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytest,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytest,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "09eefddd",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.883 means 88.3% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.326 means 32.6% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.434 means the balance between precision and recall was 43.4%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.652 means 65.2% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 63,
   "id": "cd0168dd",
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.8788447111777945\n",
      "Recall :  0.3350383631713555\n",
      "F1 score :  0.4478632478632479\n",
      "Precision :  0.6752577319587629\n"
     ]
    }
   ],
   "source": [
    "# performance analysis on train data\n",
    "ypred2 = model5.predict(xtrain)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytrain,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytrain,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytrain,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytrain,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "3d101e5d",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.878 means 87.8% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.335 means 33.5% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.447 means the balance between precision and recall was 44.7%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.675 means 67.5% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "bbd2c42f",
   "metadata": {},
   "source": [
    "## Gradient Boosting Trees"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 64,
   "id": "f8098e14",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "GradientBoostingClassifier(n_estimators=150, random_state=5)"
      ]
     },
     "execution_count": 64,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Gradient Boosting\n",
    "from sklearn.ensemble import GradientBoostingClassifier\n",
    "model6 = GradientBoostingClassifier(learning_rate=0.1,n_estimators=150,random_state=5)\n",
    "model6.fit(xtrain,ytrain)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 65,
   "id": "6a4412f0",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.9415292353823088\n",
      "Recall :  0.6630434782608695\n",
      "F1 score :  0.7577639751552795\n",
      "Precision :  0.8840579710144928\n"
     ]
    }
   ],
   "source": [
    "# performance analysis On test data\n",
    "ypred2 = model6.predict(xtest)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytest,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytest,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytest,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytest,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "093376f5",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.94 means 94% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.663 means 66.3% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.757 means the balance between precision and recall was 75.7%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.884 means 88.4% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 66,
   "id": "14b160b9",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.9643660915228808\n",
      "Recall :  0.7621483375959079\n",
      "F1 score :  0.8625180897250362\n",
      "Precision :  0.9933333333333333\n"
     ]
    }
   ],
   "source": [
    "# performance analysis on train data\n",
    "ypred2 = model6.predict(xtrain)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytrain,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytrain,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytrain,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytrain,ypred2))\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "540114ce",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.964 means 96.4% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.762 means 76.2% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.86 means the balance between precision and recall was 86%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.99 means 99% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "3ade670c",
   "metadata": {},
   "source": [
    "## XGBosst"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 67,
   "id": "a6b84b2f",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "XGBClassifier(base_score=None, booster=None, callbacks=None,\n",
       "              colsample_bylevel=None, colsample_bynode=None,\n",
       "              colsample_bytree=None, early_stopping_rounds=None,\n",
       "              enable_categorical=False, eval_metric=None, feature_types=None,\n",
       "              gamma=None, gpu_id=None, grow_policy=None, importance_type=None,\n",
       "              interaction_constraints=None, learning_rate=0.005, max_bin=None,\n",
       "              max_cat_threshold=None, max_cat_to_onehot=None,\n",
       "              max_delta_step=None, max_depth=8, max_leaves=None,\n",
       "              min_child_weight=None, missing=nan, monotone_constraints=None,\n",
       "              n_estimators=120, n_jobs=None, num_parallel_tree=None,\n",
       "              predictor=None, random_state=None, ...)"
      ]
     },
     "execution_count": 67,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "from xgboost import XGBClassifier\n",
    "model7 = XGBClassifier(learning_rate=0.005,n_estimators=120,max_depth=8)\n",
    "model7.fit(xtrain,ytrain)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 68,
   "id": "9cd4da50",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.9445277361319341\n",
      "Recall :  0.7065217391304348\n",
      "F1 score :  0.7784431137724551\n",
      "Precision :  0.8666666666666667\n"
     ]
    }
   ],
   "source": [
    "# performance analysis On test data\n",
    "ypred2 = model7.predict(xtest)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytest,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytest,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytest,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytest,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "fa197569",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.944 means 94.4% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.706 means 70.6% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.778 means the balance between precision and recall was 77.8%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.866 means 86.6% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 69,
   "id": "7ddaa8e0",
   "metadata": {
    "scrolled": false
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy :  0.9591147786946737\n",
      "Recall :  0.7442455242966752\n",
      "F1 score :  0.8422575976845152\n",
      "Precision :  0.97\n"
     ]
    }
   ],
   "source": [
    "# performance analysis on train data\n",
    "ypred2 = model7.predict(xtrain)\n",
    "print(\"Accuracy : \",metrics.accuracy_score(ytrain,ypred2))\n",
    "print(\"Recall : \",metrics.recall_score(ytrain,ypred2))\n",
    "print(\"F1 score : \",metrics.f1_score(ytrain,ypred2))\n",
    "print(\"Precision : \",metrics.precision_score(ytrain,ypred2))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "25c47795",
   "metadata": {},
   "source": [
    "Accuracy: It measures the ratio of correct predictions to the total number of predictions made. A score of 0.959 means 95.9% of the predictions made were correct.\n",
    "\n",
    "Recall: It measures the number of actual positive cases that were correctly predicted as positive. A score of 0.744 means 74.4% of the actual positive cases were correctly predicted.\n",
    "\n",
    "F1 score: It is the harmonic mean of precision and recall. It balances both precision and recall. A score of 0.84 means the balance between precision and recall was 84%.\n",
    "\n",
    "Precision: It measures the number of positive predictions that were actually correct. A score of 0.97 means 97% of the positive predictions were actually correct."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "1e646504",
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.9.12"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}
