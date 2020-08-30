---
title: '[AI] Universal Sentence Encoder & Sentence Clustering'
toc: true
toc-stick: true
search: true
categories:
  - AI
tags:
  - AI
  - Artificial Intelligence
  - NLP
  - Universal Sentence Encoder
  - Sentence Clustering
  - K-Means
excerpt: 'Universal Sentence Encoder를 사용한 문장 클러스터링을 해봅시다.'
---

# Sentence Clustering  

문장으로 구성된 Raw Data에서 의미가 비슷한 것들을 클러스터로 묶어 분류하는 작업을 진행할 예정이다.  
이 작업이 어떤 과정으로 이루어지는 필요한 개념과 도구를 알아본다.

각 문장들을 수치화하고 이 수치를 기반으로, 유사한 값들을 가진 것들끼리 묶는 과정으로 진행된다.

## DataSet 가져오기  

Dataset은 Google BigQuery Public Data Set을 이용한다.

> [Google Cloud Public Datasets](https://cloud.google.com/public-datasets/)

``` py
from google.cloud import bigquery
from google.colab import auth

auth.authenticate_user()

project_id = 'bigquery-public-data'

# # Create a "Client" object
client = bigquery.Client(project=project_id)

# Construct a reference to the "github_repos" dataset
dataset_ref = client.dataset("github_repos")

# API request - fetch the dataset
dataset = client.get_dataset(dataset_ref)

# Construct a reference to the "commits" table
table_ref = dataset_ref.table("commits")

# API request - fetch the table
table = client.get_table(table_ref)

# Extract commit message, convert to dataframe
dataframe = client.list_rows(table, 
                 selected_fields=[
                      bigquery.SchemaField("subject", "STRING")            
                 ],
                 max_results=100).to_dataframe()

commitMessages = dataframe['subject']
```  

## Embedding  

Embedding의 개념은 표현하고자 하는 대상을 dense vector 형태로 변환하는 것을 말한다. 
그 대상은 단어, 문장, 문서 전체, 이미지 등이 될 수 있다.  

여기서는 문장을 임베딩하는 Universal Sentence Encoder를 사용한다. 
입력으로 텍스트가 주어지고 출력으로는 512차원의 vector가 생성된다. 
google에서 만들었으며 tf_hub에 공개되어있다.

> [Universal Sentence Encoder](https://arxiv.org/abs/1803.11175)

``` py
!pip install tensorflow_text

import tensorflow_hub as hub
from tensorflow_text import SentencepieceTokenizer

embed = hub.load("https://tfhub.dev/google/universal-sentence-encoder-multilingual/3")
embeddings = embed(commitMessages)

print(embeddings)
```

## K-Means Clustering  

수치화된 문장 즉, 임베딩 결과를 가지고 K-Means로 클러스터링한다.

``` py
from sklearn.cluster import KMeans
import numpy as np

CLUSTER_CNT = 10

kmeans = KMeans(n_clusters = CLUSTER_CNT)
kmeans.fit(embeddings)

labels = kmeans.labels_
labelDict = dict()

# key : cluster number
# value : commit message array
for i, commitMessage in enumerate(commitMessages):
  if labels[i] in labelDict.keys():
    labelDict[labels[i]].append(commitMessage)
  else:
    labelDict[labels[i]] = [commitMessage]

for label in labelDict.keys():
  for commitMessage in labelDict[label]:
    print('{} : {}'.format(label, commitMessage))
```  

아래는 100개의 메시지를 10개의 클러스터로 분류한 결과이다.

``` txt
0 : Webtree doc from Marc
0 : Merge branch 'chef-vendor-apt'
0 : Changed parsers to make is compatible with new branch. 
0 : Merge branch '1.0'
0 : Merge branch 'google'
0 : Merge commit for internal changes
0 : Merge branch 'devel'
0 : Merge branch 'master' into api-refactor
0 : Merge rbenv parent repository
0 : Merge branch 'master' into profile_deletion
0 : Merge branch 'hotfix/4284' into develop
0 : Merge branch 'version/bump' into develop
3 : packages/liboil: fix whitespaces
3 : Stop supporting kUnknown_BmpHeaderType DO NOT MERGE am: 71eb09f952 am: 63762a3d4f am: 6bcf8398db am: fb82d6bf7c am: ee560f7102 am: 47dfb6b44c  -s ours
3 : [arm-fast-isel] Doublewords only require word-alignment. rdar://10528060
3 : Fix access of MFT_RECORD->bytes_in_use to use le32_to_cpu() instead of le16_to_cpu() in libntfs/volume.c.  (Pete Curran)
3 : Merge changes I85492417,I93389a2c into nougat-cts-dev am: 37eaa88ac2 am: a7dc5ef89b
3 : Merged V4.1-BUG-FIX to HEAD (RECORD ONLY)    38419: Merged HEAD to BRANCHES/DEV/V4.1-BUG-FIX:       38381: ALF-14835: Sharepoint/WebDAV: exclusive lock fails deemed to be owned by someone else.
3 : DO NOT MERGE ANYWHERE Update support library version to 24.1.0 am: 95003af5cc  -s ours am: f705afce0e  -s ours
3 : Merge branch \\'ub-launcher3-burnaby-nyc\\' into nyc-dev b/29535844 am: 3c0c2c4b02 am: 027594fc1b am: 9015be72e4 am: a088d2be66
3 : Merged HEAD-BUG-FIX (4.3/Cloud) to HEAD (4.3/Cloud)    65156: Merged AUTO1 (4.3/Cloud) to HEAD-BUG-FIX (4.3/Cloud)       63143: WEBDRONE-580
3 : builtin-clone: use strbuf in clone_local() and copy_or_link_directory()
3 : Fix <net/if.h> compile time problems on OpenBSD for good
3 : Merge branch 'update-code-package' of git://github.com/neeckeloo/zf2 into hotfix/3297
3 : Merge branch 'ZF-4817' of https://github.com/thomasweidner/zf2 into hotfix/zf-4817
3 : Merge branch 'ZF2-512' of https://github.com/mediacabinet/zf2 into hotfix/zf2-512
3 : Merge branch 'hotfix/ZF-9521' of https://github.com/adamlundrigan/zf2 into hotfix/zf-9521
3 : Merge branch 'hotfix/codegenerator-clean' of https://github.com/Maks3w/zf2 into hotfix/code-generator-cleanup
3 : Merge branch 'hotfix/ZF-11036' of https://github.com/thomasweidner/zf2 into hotfix/zf-11036
4 : Fix an assertion hit when the serialized diagnostics writer receive a diagnostic from the frontend when the location is invalid and the SourceManager null.
4 : - Fix the release number computation and add some additional consistency checks.
4 : Close #5169 - session already invalidated
4 : Changes to fix possible bug.
4 : Revert "ensure auto skipping of db tests is verboten"
4 : develop - fixed test issue
4 : Revert: Reorganize sidebar according to mockups
4 : qemu: unittest: Tweak log message
6 : [NA] Обновление exportIntegration.config, добавился Каменск-Уральский (109)
6 : [ERM-3133] Core Main => Prerelease FI
6 : [NA] Core Main RI
6 : [arm fast-isel] Appease the machine verifier by using the proper register classes. rdar://12719844
6 : [NA] 2.1-E12-ThreeLittlePigs
6 : [NA] Актуализация Prerelease -> Main RI
6 : [ERM-5249], fix, 8: валюта в ПФ
7 : listo ?3
7 : force commit
7 : build bump
7 : Cleanup
7 : Pass tenant_name to zuul config.
8 : Bug 53480 - Add Kerberos support to Http Sampler (HttpClient4) svn eol Bugzilla Id: 53480
8 : Update svn:ignore.
8 : New script needs python-yaml.
8 : Remove empty directory.
8 : update vendor
8 : update generated files
8 : tag release.2010-11-23
8 : Updated FAST_Library VisualStudio project
9 : CLOSE #4008 - trunk(7.2.x) - Add a local build feature to Convertigo Studio  - Enable Cordova "Local build" menu
9 : inprogress #4547 - Fixed CustomTransaction URI
9 : FI [$/ERM.BL/Dev/Features/ActivityMigration/BL]
9 : inprogress #3384 - (trunk) handled objects like arrays in compact mode
9 : inprogress #2583 - added the ant build procedure in trunk
9 : Merge pull request #17373 from Microsoft/10662-volumetests
9 : Merge pull request #228 from phobson/no-filters-on-db-object
9 : Merge pull request #63 from twhittock/const-cast
9 : [flang] Merge pull request flang-compiler/f18#319 from flang-compiler/eas0
9 : Merge pull request #572 from icgc-dcc/feature/DCC-1292-portal-api-expose-static-counts
9 : Merge pull request #13833 from Microsoft/10662-dockerbuild
9 : Merge pull request #1469 from arkpar/mix_test
9 : Merge pull request zendframework/zf2#6246 from rodmcnew/6244-translation-loaders-from-config
9 : Merge pull request zendframework/zf2#5242 from dpommeranz/fix/form-collection-attributes
2 : Initialized merge tracking via "svnmerge" with revisions "1-11870" from  https://opennms.svn.sourceforge.net/svnroot/opennms/opennms/branches/feature-hibernate-merge
2 : Merge root@wimsey:/home/donp/dev/working/setools-dev into twoface.(none):/home/method/temp/setools-release
2 : Merge "[RenderScript] Add finalizer to support lib context." am: b0348c2928 am: afa1917182 am: bc46c8c574
2 : Merge branch 'issue-464-reql-admin-api' of github.com:rethinkdb/docs into issue-464-reql-admin-api
2 : Add 'src/WellCommerce/Bundle/CoreBundle/' from commit '2712d9fcba63b4b9f9f9c9de21f88423027a656c'
2 : Merge mysql.com:/home/jimw/my/mysql-4.0-clean into mysql.com:/home/jimw/my/mysql-4.1-clean
2 : Merge branch 'time_saving' of https://github.com/arbalet-project/arbasdk into time_saving
2 : Merge remote-tracking branch 'origin/issue/186_cannot_read_property_id_of_undefined'
2 : Merge jcole@work.mysql.com:/home/bk/mysql-4.0 into tetra.spaceapes.com:/home/jcole/bk/mysql-4.0
2 : Merge branch 'feature/cache/xcache' of git://github.com/marc-mabe/zf2 into feature/xcache
2 : Merge branch 'session-renamed-interfaces' of https://github.com/prolic/zf2 into feature/zen27-session
1 : Merge branch 'master' of https://github.com/senbox-org/s1tbx
1 : Merge branch 'master' of github.com:concordusapps/grunt-haml
1 : Merge branch 'master' of github.com:stevesims/vptimelinehandler
1 : Merge remote-tracking branch 'refs/remotes/origin/greenkeeper-babel-plugin-transform-es2015-modules-commonjs-6.16.0'
1 : Merge branch 'master' of github.com:Locbit/OctoPrint-Locbit
1 : Merge remote-tracking branch 'prolic/view' into prolic-zen-52
1 : Merge remote-tracking branch 'origin/master'
1 : Merge branch 'devel' of github.com:ska-sa/casperfpga into devel
1 : Merge branch 'develop' of https://github.com/SimonHarte/SpartanGrid into develop
1 : Merge branch 'master' of https://github.com/TECLIB/geninventorynumber
1 : Merge remote-tracking branch 'origin/master' into frameworks
1 : Merge branch 'master' of https://github.com/CMPUT301W15T02/TeamTo
1 : Merge branch 'master' of http://github.com/Tamarabyte/cmput410-project
1 : Merge branch 'master' of https://github.com/fatih/structure
1 : Merge branch 'master' of https://github.com/KrazyTheFox/StarDB-for-Java
1 : Merge branch 'master' of github.com:WGEN-SLI/SLI
5 : (portal-ui) fixes merge issues
5 : (portal-ui) small tweaks, update team
``` 

## Elbow Method  

K-Means의 한계점으로는 클러스터 개수 선정에 있다.  
위의 예제에서는 10개의 클러스터로 분류를 하도록 진행하였는데, 
적절한 클러스터 개수를 어느정도 파악할 수 있는 elbow method을 적용해본다.  

``` py
from sklearn.cluster import KMeans
from matplotlib import pyplot as plt

sse = []
for cluster_cnt in range(1, 20):
  kmeans = KMeans(n_clusters=cluster_cnt, init='k-means++', random_state=0)
  kmeans.fit(embeddings)
  sse.append(kmeans.inertia_)

plt.plot(range(1, 20), sse, marker='o')
plt.xlabel('cluster cnt')
plt.ylabel('SSE')
plt.show()
```

![elbow_method](/assets/images/ai/elbow_method.png)  

그래프가 많이 꺾이면서 이 후 부터는 값의 변화가 거의 없는 지점을 elbow라고 한다. 
이 근처의 값을 선택하는 것이 어느정도 최적화 된 클러스터의 수라고 할 수 있다.