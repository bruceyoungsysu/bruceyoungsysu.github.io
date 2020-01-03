---
title: Santander Customer Transaction Prediction
categories:
 - Kaggle
 - LightGBM
 - Recognition
tags: Regression
image: https:https://s3-ap-south-1.amazonaws.com/av-blog-media/wp-content/uploads/2017/06/11194227/depth.png

---

This is a kaggle chanllenge launched by Santander to help looking for ways to help customers understand their financial health and identify which products and services might help them achieve their monetary goals. On the other hand, the chanllenge is about to help identify which customers will make a specific transaction in the future, irrespective of the amount of money transacted. 

### Data Investigation

The training set of the data is pretty simple. Each record in the dataset represents a single record of transaction. For each record, there is a field of `ID_code` to identify each record. And another field provides the target of training. There are also 98 parameters which are named as `var0` to `var97` with no extra information about them provided. All of the 98 parameters are nearly in normal distribution.

### Light GBM

In this project we use Light GBM which is a fast, distributed, high-performance gradient boosting framework based on decision tree algorithm, used for ranking, classification and many other machine learning tasks.

Since it is based on decision tree algorithms, it splits the tree leaf wise with the best fit whereas other boosting algorithms split the tree depth wise or level wise rather than leaf-wise. So when growing on the same leaf in Light GBM, the leaf-wise algorithm can reduce more loss than the level-wise algorithm and hence results in much better accuracy which can rarely be achieved by any of the existing boosting algorithms. Also this algorithm is surprisingly fast. (https://www.analyticsvidhya.com/blog/2017/06/which-algorithm-takes-the-crown-light-gbm-vs-xgboost/)

For more detailed explanation of Light BGM and XGBOOST please refer to this [site](https://www.analyticsvidhya.com/blog/2017/06/which-algorithm-takes-the-crown-light-gbm-vs-xgboost/).

Our lightgbm model is trained with the following parameter:

```
param = {
    'bagging_freq': 5,
    'bagging_fraction': 0.335,
    'boost_from_average':'false',
    'boost': 'gbdt',
    'feature_fraction': 0.041,
    'learning_rate': 0.0083,
    'max_depth': -1,
    'metric':'auc',
    'min_data_in_leaf': 80,
    'min_sum_hessian_in_leaf': 10.0,
    'num_leaves': 13,
    'num_threads': 8,
    'tree_learner': 'serial',
    'objective': 'binary', 
    'verbosity': -1
}
```

### Data Augmentation

we also utilized the data augmentation method to create more records, but minor improvements are made by this method.

```
# data augmentation
# shuffle every column respectively to create more data 
def augment(train,num_n=1,num_p=2):
    new_train=[train]
    
    n=train[train.target==0]
    for i in range(num_n):
        new_train.append(n.apply(lambda x:x.values.take(np.random.permutation(len(n)))))
    
    for i in range(num_p):
        p=train[train.target>0]
        new_train.append(p.apply(lambda x:x.values.take(np.random.permutation(len(p)))))
    return pd.concat(new_train)
```

There are also many other ways to do feature engineering to add data to each record. The first thing to do is to round the decimal of each number in each parameter, it can be done by rounding to 1 or 2 decimals:

```
# round decimal
features = [col for col in df_train.columns if col not in ["target"]]
for feature in features:
    df_train[feature+"_r2"] = np.round(df_train[feature],2)
    df_train[feature+"_r1"] = np.round(df_train[feature],1)
    df_test[feature+"_r2"] = np.round(df_test[feature],2)
    df_test[feature+"_r1"] = np.round(df_test[feature],1)
```

Another way to do feature engineering is to add statistical data for each record such as the max or the mean value among all 98 parameters:

```
idx = df_train.columns.values[1:]
for df in [df_train, df_test]:
    df['sum'] = df[idx].sum(axis=1)
    df['min'] = df[idx].min(axis=1)
    df['max'] = df[idx].max(axis=1)
    df['mean'] = df[idx].mean(axis=1)
    df['std'] = df[idx].std(axis=1)
    df['skew'] = df[idx].std(axis=1)
    df['kurt'] = df[idx].kurt(axis=1)
    df['med'] = df[idx].median(axis=1)
```

The training part is straight forward as we trained the model for 1000000 rounds with early stoppping at round of 4000. We also used 10 fold :

```
predictions = np.zeros(len(df_test))
for fold_, (trn_idx,val_idx) in enumerate(skf.split(df_train, df_train.target)):
    print('fold: {}'.format(fold_))
    t = df_train.iloc[trn_idx]
    t = augment(t)
    trn_dataset = lgb.Dataset(t.drop(columns="target"),label=t.target)
    val_dataset = lgb.Dataset(df_train.iloc[val_idx].drop(columns="target"),label=df_train.iloc[val_idx].target)
    num_round = 1000000
    clf = lgb.train(param,trn_dataset,num_round,valid_sets=[trn_dataset,val_dataset],verbose_eval=1000,early_stopping_rounds=4000)
#     oof[val_idx] = clf.predict(df_train.iloc[val_idx],num_iteration=clf.best_iteration)
    predictions += clf.predict(df_test,num_iteration=clf.best_iteration)/10 #add 10 folds together
```

Finally the result could give an accuracy on public test dataset of 0.90107 and accuracy of 0.89993 on private dataset. Also I could get my first bronze metal with the result in this competition:

![](https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/santander/Screen Shot 2020-01-03 at 16.26.15.png){:width="200px"}

