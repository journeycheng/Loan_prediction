# Loan_prediction
基于提供的申请信息，自动判断用户是否有贷款的资格
```python
train_df = pd.read_csv('train.csv')
print train_df.info()

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 614 entries, 0 to 613
Data columns (total 13 columns):
Loan_ID              614 non-null object
Gender               601 non-null object
Married              611 non-null object
Dependents           599 non-null object
Education            614 non-null object
Self_Employed        582 non-null object
ApplicantIncome      614 non-null int64
CoapplicantIncome    614 non-null float64
LoanAmount           592 non-null float64
Loan_Amount_Term     600 non-null float64
Credit_History       564 non-null float64
Property_Area        614 non-null object
Loan_Status          614 non-null object
dtypes: float64(4), int64(1), object(8)
memory usage: 62.4+ KB
```
> - Loan_ID: 唯一贷款ID
> - Gender: 性别。Male/Female
> - Married: 婚否。Y/N
> - Dependents: 受抚养者数量
> - Education: Graduate（大学毕业生，研究生）/ Under Graduate(本科生)
> - Self_Employed: 个体经营者。Y/N
> - ApplicationIncome: 收入
> - CoapplicationIncome: 
> - LoanAmount: 贷款额度(千)
> - Loan_Amount_Term: 贷款期限（月）
> - Credit_History: credit history meets guidelines.0和1
> - Property_area: 房产位置。Urban/Semi Urban/Rural
> - Loan_status: 是否同意贷款。Y/N

Partners in a transaction will use co-applicant status to share the responsibility of a loan as well as the benefits of ownership for the product purchased with the loan. Co-applicants legally agree to share the property and the responsibility for repayment of the loan

```python
print train_df.isnull().sum()

Loan_ID               0
Gender               13
Married               3
Dependents           15
Education             0
Self_Employed        32
ApplicantIncome       0
CoapplicantIncome     0
LoanAmount           22
Loan_Amount_Term     14
Credit_History       50
Property_Area         0
Loan_Status           0
dtype: int64
```

```python
print train_df.head()
    Loan_ID Gender Married Dependents     Education Self_Employed  \
0  LP001002   Male      No          0      Graduate            No   
1  LP001003   Male     Yes          1      Graduate            No   
2  LP001005   Male     Yes          0      Graduate           Yes   
3  LP001006   Male     Yes          0  Not Graduate            No   
4  LP001008   Male      No          0      Graduate            No   

   ApplicantIncome  CoapplicantIncome  LoanAmount  Loan_Amount_Term  \
0             5849                0.0         NaN             360.0   
1             4583             1508.0       128.0             360.0   
2             3000                0.0        66.0             360.0   
3             2583             2358.0       120.0             360.0   
4             6000                0.0       141.0             360.0   

   Credit_History Property_Area Loan_Status  
0             1.0         Urban           Y  
1             1.0         Rural           N  
2             1.0         Urban           Y  
3             1.0         Urban           Y  
4             1.0         Urban           Y 
```
测试集：
```python
test_df = pd.read_csv('test.csv')
print test_df.info()

RangeIndex: 367 entries, 0 to 366
Data columns (total 12 columns):
Loan_ID              367 non-null object
Gender               356 non-null object
Married              367 non-null object
Dependents           357 non-null object
Education            367 non-null object
Self_Employed        344 non-null object
ApplicantIncome      367 non-null int64
CoapplicantIncome    367 non-null int64
LoanAmount           362 non-null float64
Loan_Amount_Term     361 non-null float64
Credit_History       338 non-null float64
Property_Area        367 non-null object
dtypes: float64(3), int64(2), object(7)
memory usage: 34.5+ KB
```

```python
print test_df.isnull().sum()

Loan_ID               0
Gender               11
Married               0
Dependents           10
Education             0
Self_Employed        23
ApplicantIncome       0
CoapplicantIncome     0
LoanAmount            5
Loan_Amount_Term      6
Credit_History       29
Property_Area         0
dtype: int64
```

```python
print test_df.head()

    Loan_ID Gender Married Dependents     Education Self_Employed  \
0  LP001015   Male     Yes          0      Graduate            No   
1  LP001022   Male     Yes          1      Graduate            No   
2  LP001031   Male     Yes          2      Graduate            No   
3  LP001035   Male     Yes          2      Graduate            No   
4  LP001051   Male      No          0  Not Graduate            No   

   ApplicantIncome  CoapplicantIncome  LoanAmount  Loan_Amount_Term  \
0             5720                  0       110.0             360.0   
1             3076               1500       126.0             360.0   
2             5000               1800       208.0             360.0   
3             2340               2546       100.0             360.0   
4             3276                  0        78.0             360.0   

   Credit_History Property_Area  
0             1.0         Urban  
1             1.0         Urban  
2             1.0         Urban  
3             NaN         Urban  
4             1.0         Urban
```

train 和test合并
```python
df = pd.concat([train_df, test_df], axis=0)
train_size = train_df.shape[0]
```
### Married
```python
print df[df.Married.isnull()][['Gender', 'Dependents','Self_Employed', 'Education', 'ApplicantIncome']]

     Gender Dependents Self_Employed Education  ApplicantIncome
104    Male        NaN            No  Graduate             3816
228    Male        NaN            No  Graduate             4758
435  Female        NaN            No  Graduate            10047
```
和结婚信息最息息相关的Dependents竟然也是NaN，吐血，只能根据其它的字段信息进行推断了

```python
facet = sns.FacetGrid(df[(df.Self_Employed == 'No') & (df.Gender=='Female') & (df.Education == 'Graduate') & (df.ApplicantIncome > 10000)], hue='Married', aspect=4)
facet.map(sns.kdeplot, 'ApplicantIncome', shade=True)
facet.set(xlim=(10000, df[(df.Self_Employed == 'No') & (df.Gender=='Female') & (df.Education == 'Graduate') & (df.ApplicantIncome > 10000)]['ApplicantIncome'].max()))
facet.add_legend()
plt.title('Female and Income > 10000')
plt.show()
```
![](raw/figure_4.png?raw=true)

从上图来看，女性申请者，收入略多于10000的非常有可能是未婚的。

```python
sns.countplot(x='Married', hue='Gender', data=df[(df.Self_Employed == 'No') & (df.Education == 'Graduate') & (df.ApplicantIncome > 10000)])
plt.title('Income > 10000')
plt.show()
```
![](raw/figure_3.png?raw=true)

从上图中可以看出,Female申请额度大于10000时，没结婚的偏多。

综合以上两点，该女性很可能是未婚的。

```python
facet = sns.FacetGrid(df[(df.Self_Employed == 'No') & (df.Education == 'Graduate') & (df.ApplicantIncome < 5000)], hue='Married', aspect=4)
facet.map(sns.kdeplot, 'ApplicantIncome', shade=True)
facet.set(xlim=(0, df[(df.Self_Employed == 'No') & (df.Education == 'Graduate') & (df.ApplicantIncome < 5000)]['ApplicantIncome'].max()))
facet.add_legend()
plt.show()
```
![](raw/figure_5.png?raw=true)

Male申请者，未婚和已婚的收入分布基本一致。

```python
sns.countplot(x='Married', hue='Gender', data=df[(df.Self_Employed == 'No') & (df.Education == 'Graduate') & (df.ApplicantIncome < 5000)])
plt.title('Income < 5000')
plt.show()
```
![](raw/figure_2.png?raw=true)

但是从统计上来看，已婚的男士较多。

综上两点，该两位男性很有可能是已婚的。

所以：
```python
_ = df.set_value((df.ApplicantIncome > 10000) & (df.Married.isnull()), 'Married', 'No')
_ = df.set_value((df.ApplicantIncome < 5000) & (df.Married.isnull()), 'Married', 'Yes')
```

### Gender

查看性别对贷款的影响
```python
sns.countplot(x='Gender', hue='Loan_Status', data = train_df)
plt.show()
```
![](raw/figure_1.png?raw=true)

产看Gender缺失值的其它特征：
```python
print df[df.Gender.isnull()][['Self_Employed', 'ApplicantIncome']]

    Self_Employed  ApplicantIncome
23             No             3365
126            No            23803
171            No            51763
188           Yes              674
314            No             2473
334           Yes             9833
460           Yes             2083
467            No            16692
477            No             2873
507            No             3583
576            No             3087
588            No             4750
592           Yes             9357
22             No             3909
51             No             3500
106            No             1596
138            No             3333
209            No             2038
231           Yes             2860
245            No             3186
279            No            29167
296            No             6478
303           Yes              570
318            No             4768
```

```python
sns.boxplot(x='Gender', y='ApplicantIncome', hue='Self_Employed', data=df)
plt.axhline(y='5000', color='red')
plt.axhline(y='10000', color='green')
plt.show()
```
![](raw/figure_9.png?raw=true)

对Gender进行预测。

选择相关的不含缺失值的字段用于预测。
```python
df_for_gender = df[['Gender', 'Married', 'Property_Area', 'Education', 'ApplicantIncome']]
```

对ApplicantIncome字段进行缩放处理
```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
df_for_gender['Norm_Income'] = pd.Series(scaler.fit_transform(df_for_gender['ApplicantIncome'].reshape(-1,1)).reshape(-1), index=df_for_gender.index)
df_for_gender.drop('ApplicantIncome', axis=1, inplace=True)
```

对类别型变量进行dummies处理
```python
df_for_gender = pd.get_dummies(df_for_gender, columns=['Married', 'Property_Area', 'Education'])
```

经过上面的步骤，用于预测Gender的数据集如下：
```python
print df_for_gender.info()

Data columns (total 9 columns):
Gender                     957 non-null object
Norm_Income                981 non-null float64
Married_No                 981 non-null float64
Married_Yes                981 non-null float64
Property_Area_Rural        981 non-null float64
Property_Area_Semiurban    981 non-null float64
Property_Area_Urban        981 non-null float64
Education_Graduate         981 non-null float64
Education_Not Graduate     981 non-null float64
dtypes: float64(8), object(1)
memory usage: 76.6+ KB
```

训练逻辑回归模型

```python
from sklearn.linear_model import LogisticRegression
lg = LogisticRegression()
parameters={'C': [0.5]}
clf_lgl = get_model(lg, parameters, X_train, y_train, scoring)
print clf_lgl

LogisticRegression(C=0.5, class_weight=None, dual=False, fit_intercept=True,
          intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
          penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
          verbose=0, warm_start=False)
```
模型测试正确率还算可以。
```python
print accuracy_score(y_train, clf_lgl.predict(X_train))

0.811659192825
```

CV检验：
```python
plot_learning_curve(clf_lgl, 'Logistic Regression', X, y, cv=4)
plt.show()
```
![](raw/figure_10.png?raw=true)

模型正常。

对缺失值进行预测，并替换缺失值
```python
y_predict = clf_lgl.predict(X_predict)
df.set_value(df.Gender.isnull(), 'Gender', y_predict)
```

删除后面不用的变量
```python
del df_for_gender, X, y, X_train, X_test, y_train, y_test, X_predict, y_predict, lg, clf_lgl
```
### Dependents
```python
sns.countplot(x='Dependents', hue ='Loan_Status', data=df)
plt.title('count by Dependents and Loan_Status')
plt.show()
```
![](raw/figure_11.png?raw=true)

对Dependents进行类似的预测，但是正确率很低。
```python
df_for_dependents = df[['ApplicantIncome', 'Education', 'Gender', 'Married', 'Property_Area', 'Dependents']]
```

![](raw/figure_12.png?raw=true)

预测的准确率有点低

### Credit_history
```python
LogisticRegression(C=0.5, class_weight=None, dual=False, fit_intercept=True,
          intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
          penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
          verbose=0, warm_start=False)
0.835182250396
```

![](raw/figure_13.png?raw=true)


### Self_Employed
```python
from sklearn.svm import SVC
svc = SVC(random_state=42, kernel='poly', probability=True)
parameters = {'C': [35], 'gamma': [0.0055], 'coef0': [0.1],
              'degree':[2]}
clf_svc = get_model(svc, parameters, X_train, y_train, scoring)
print (clf_svc)
print (accuracy_score(y_test, clf_svc.predict(X_test)))
plot_learning_curve(clf_svc, 'SVC for Self_Employed Precidtion', X, y, cv=4);
plt.show()


SVC(C=35, cache_size=200, class_weight=None, coef0=0.1,
  decision_function_shape=None, degree=2, gamma=0.0055, kernel='poly',
  max_iter=-1, probability=True, random_state=42, shrinking=True,
  tol=0.001, verbose=False)
0.852517985612
```
![](raw/figure_14.png?raw=true)


### LoanAmount
```python
df['TotalIncome'] = df['ApplicantIncome'] + df['CoapplicantIncome']
sns.FacetGrid(df, hue='Loan_Status', size=5).map(plt.scatter, 'TotalIncome','LoanAmount').add_legend()
plt.title('LoanAmount and TotalIncome')
plt.show()
```
![](raw/figure_15.png?raw=true)

分布毫无规律可言。

加上其它的字段信息进行线性回归。

先采用随机值处理

### Loan_Amount_Term
```python
df['Total_Income'] = df['ApplicantIncome'] + df['CoapplicantIncome']

print df[df.Loan_Amount_Term.isnull()][['LoanAmount', 'Total_Income']]

     LoanAmount  Total_Income
19        115.0        6100.0
36        100.0        3158.0
44         96.0        4695.0
45         88.0        3410.0
73         95.0        4755.0
112       152.0        7686.0
165       182.0        6873.0
197       120.0        4272.0
223       175.0        8588.0
232       120.0        5787.0
335        70.0        9993.0
367       124.0        5124.0
421        80.0        2720.0
423       110.0        8917.0
45        185.0        8160.0
48        187.0       10130.0
117        80.0        4416.0
129        70.0        5409.0
184       150.0       10916.0
214       118.0        7473.0
```

```python
print df['Loan_Amount_Term'].value_counts()

360.0    823
180.0     66
480.0     23
300.0     20
240.0      8
84.0       7
120.0      4
60.0       3
36.0       3
12.0       2
350.0      1
6.0        1
```
银行贷款时间一般都是一些固定的。
```python
sns.countplot(x='Loan_Amount_Term', hue='Loan_Status', data=df)
plt.show()
```
![](raw/figure_16.png?raw=true)

```python

LogisticRegression(C=0.5, class_weight=None, dual=False, fit_intercept=True,
          intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
          penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
          verbose=0, warm_start=False)
0.830508474576
```
![](raw/figure_17.png?raw=true)

到这里数据集基本完成了缺失值的替换。

```python
print df.info()

Int64Index: 981 entries, 0 to 366
Data columns (total 14 columns):
ApplicantIncome      981 non-null int64
CoapplicantIncome    981 non-null float64
Credit_History       981 non-null float64
Dependents           981 non-null object
Education            981 non-null object
Gender               981 non-null object
LoanAmount           981 non-null float64
Loan_Amount_Term     981 non-null object
Loan_ID              981 non-null object
Loan_Status          614 non-null object
Married              981 non-null object
Property_Area        981 non-null object
Self_Employed        981 non-null object
Total_Income         981 non-null float64
dtypes: float64(4), int64(1), object(9)
memory usage: 115.0+ KB
```

逻辑回归模型：

```python
LogisticRegression(C=0.5, class_weight=None, dual=False, fit_intercept=True,
          intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
          penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
          verbose=0, warm_start=False)
0.789189189189
```
![](raw/figure_18.png?raw=true)

```python
RandomForestClassifier(bootstrap=True, class_weight=None, criterion='entropy',
            max_depth=None, max_features='auto', max_leaf_nodes=None,
            min_samples_leaf=12, min_samples_split=5,
            min_weight_fraction_leaf=0.0, n_estimators=500, n_jobs=1,
            oob_score=True, random_state=42, verbose=0, warm_start=False)
```

## Ensemble

```python
from sklearn.ensemble import VotingClassifier
clf_vc = VotingClassifier(estimators=[('lgl', clf_lgl), ('rfc1', clf_rfc1)], 
                          voting='hard', weights=[1,1])
clf_vc = clf_vc.fit(X_train, y_train)

print (accuracy_score(y_test, clf_vc.predict(X_test)))
print (clf_vc)

0.789189189189
VotingClassifier(estimators=[('lgl', LogisticRegression(C=0.5, class_weight=None, dual=False, fit_intercept=True,
          intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
          penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
          verbose=0, warm_start=False)), ('rfc1', Rand...stimators=500, n_jobs=1,
            oob_score=True, random_state=42, verbose=0, warm_start=False))],
         voting='hard', weights=[2, 1])
```
