---
title: ".gitlab-ci.ymlã®ä¿ºçTips"
emoji: "ð¦"
type: "tech"
topics: [GitLab, DevOps, CI/CD]
published: true
published_at: 2018-12-09 12:48
---

# ã¯ããã«

GitLab ãä½¿ãã¯ããã¦ 1 å¹´åã«ãªãã¾ããã`.gitlab-ci.yml` ã«é¢ãããã¦ãã¦ãã»ã©ã»ã©ã«æºã¾ã£ã¦ããæ°ãããã®ã§ Tips ã¨ãã¦ã¾ã¨ãã¦ã¿ã¾ããã

**æ³¨æ**

- Web ã¢ããªã±ã¼ã·ã§ã³ã® CI/CD ã«é¢ãã Tips ãå¤ãã§ãã
- `.gitlab-ci.yml` ã®ä¾ã¯ Python/Django ã§æ¸ãã¦ãã¾ãããç¹ã«é£ãããã¨ã¯æ¸ãã¦ããªãã¤ããã§ããé©å®å¾æãªè¨èªã«ç½®ãæãã¦èª­ãã§ãã ããã
- æ¬è¨äºã§ã¯[å¬å¼ãã­ã¥ã¡ã³ã](https://docs.gitlab.com/ce/ci/yaml/)ã«å£ãã `.gitlab-ci.yml` ã®ä¸ã¤ã®å®è¡åä½ã `ã¸ã§ã`ãã¸ã§ãã«å®ç¾©ãã `image` ã `stage`, `script` ãªã©ã®ã¸ã§ãã®ãµãã¾ãã `ã­ã¼ã¯ã¼ã` ã¨å¼ã³ã¾ãã

# Index

1. [Services](https://docs.gitlab.com/ee/ci/services/README.html)ï¼ã³ã³ããã¤ã¡ã¼ã¸ãè¤æ°æ±ã
2. [Anchors](https://docs.gitlab.com/ee/ci/yaml/README.html#anchors)ï¼ã¸ã§ãã®ãã³ãã¬ã¼ãã§ã¹ãªã ãª YAML ã«
3. [Predefined environment variables](https://docs.gitlab.com/ee/ci/variables/README.html#predefined-environment-variables)ï¼ç¨æããã¦ãã GitLab CI/CD å¤æ°ãæ´»ç¨ãã
4. [Only and Except](https://docs.gitlab.com/ce/ci/yaml/#onlyexcept-basic)ï¼å¶éä»ãã®ã¸ã§ããä½æãã
5. [when:manual](https://docs.gitlab.com/ee/ci/yaml/README.html#whenmanual)ï¼å®å¿å®å¨ãªããã­ã¤
6. [artifacts](https://docs.gitlab.com/ee/user/project/pipelines/job_artifacts.html)ï¼ã«ãã¬ãã¸ã¬ãã¼ããå¯è¦åãã

# Tips

## 1. Servicesï¼ã³ã³ããã¤ã¡ã¼ã¸ãè¤æ°æ±ã

https://docs.gitlab.com/ee/ci/services/README.html

`services` ã¨ããã­ã¼ã¯ã¼ããç¨ããã¨ããã¼ã¹ã¤ã¡ã¼ã¸ã¨æ¥ç¶å¯è½ãªã³ã³ãããå®ç¾©ã§ãã¾ãã
Docker in Docker ã§ã³ã³ããã¬ã¸ã¹ããªã« Docker ã¤ã¡ã¼ã¸ãããã·ã¥ããããåä½ãã¹ãã§ãã¼ã¿ãã¼ã¹ã³ã³ããã«æ¥ç¶ãããããã¨ãã£ãã±ã¼ã¹ã«æç¨ã§ãã

ä¾ãã°ãä¸è¨ã®ããã«å®ç¾©ãããã¨ã§ `postgres` ã¨ãããã¹ãåã§ PostgreSQL ã«æ¥ç¶ã§ãã¾ãã

```yaml
services:
  - postgres:latest
variables:
  POSTGRES_DB: custom_db
  POSTGRES_USER: custom_user
  POSTGRES_PASSWORD: custom_password
```

PostgreSQL ã®å¬å¼ã¤ã¡ã¼ã¸ã§ç¨æããã¦ããç°å¢å¤æ°ã«ãã¼ã¿ãã¼ã¹åãã¦ã¼ã¶åããã¹ã¯ã¼ããè¨­å®ãã¾ãã

ç°å¢å¤æ°ã«é¢ããè©³ç´°ã¯ PostgreSQL ã® DockerHub ãåç§ãã¦ãã ããã  
FYI: https://hub.docker.com/_/postgres/

`DATABASE_URL` ã§æ¥ç¶æå ±ãè¡¨ãã¨ã `postgres://$POSTGRES_USER:$POSTGRES_PASSWORD@postgres:5432/$POSTGRES_DB` ã¨ãªãã¾ãã

## 2. Anchorsï¼ã¸ã§ãã®ãã³ãã¬ã¼ãã§ã¹ãªã ãª YAML ã«

https://docs.gitlab.com/ee/ci/yaml/README.html#anchors

Anchors ã¨ããæ©è½ãå©ç¨ãã¦ã¸ã§ãã®ãã³ãã¬ã¼ããä½æãããã¨ã§ã`.gitlab-ci.yml` ãã¹ãªã ã«ä¿ã¤ãã¨ãã§ãã¾ãã

ä¾ãã°ãtest1, test2 ã¨ãããã¹ããããããå®è¡ãããã¨ãã

```yaml
test1:
  stage: test
  image: python:latest
  before_script:
    - pipenv install --dev --system
  script:
    - pytest test1

test2:
  stage: test
  image: python:latest
  before_script:
    - pipenv install --dev --system
  script:
    - pytest test2
```

ä¸è¨ã®ããã«æ¸ããã¨ãã§ãã¾ãããAnchors ãå©ç¨ããã¨ä¸è¨ã®ããã«æ¸ããã¨ãã§ãã¾ãã

```yaml
.test_template: &test_definition
  stage: test
  image: python:latest
  before_script:
    - pipenv install --dev --system

test1:
  <<: *test_definition
  script:
    - pytest test1

test2:
  <<: *test_definition
  script:
    - pytest test2
```

ä»åã®ä¾ã§ã¯è¡æ°çãªå·®ããã¾ãåºã¾ããã§ããããã¸ã§ããå¢ããã°å¢ããã»ã© Anchors ã®å¹æãé«ã¾ãã¾ãã

## 3. Predefined environment variablesï¼ç¨æããã¦ãã GitLab CI/CD å¤æ°ãæ´»ç¨ãã

https://docs.gitlab.com/ee/ci/variables/README.html#predefined-environment-variables

`.gitlab-ci.yml` ã«ã¯ GitLab ããããããç¨æãã¦ããç°å¢å¤æ°ãããã¤ãããã¾ãã
ããããæ´»ç¨ãããã¨ã§ä½è¨ãªç°å¢å¤æ°å®ç¾©ãããªãã¦æ¸ãã ããå¥ã®ãã­ã¸ã§ã¯ãã§ãã®ã¾ã¾æ´»ç¨ã§ããããã¾ãã

ä¾ãã°ãGitLab ã³ã³ããã¬ã¸ã¹ããªã¸ã®ã¤ã¡ã¼ã¸ã®ããã·ã¥ã§ Predefined environment variables ãæ´»ç¨ãããã¨ãã§ãã¾ãã

```yaml
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
  script:
    - docker build --pull -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
  only:
    - tags
```

`CI_JOB_TOKEN`, `CI_REGISTRY`, `CI_REGISTRY_IMAGE`, `CI_COMMIT_TAG` ã¯ Predefined variables ã§ãã

## 4. Only and Exceptï¼å¶éä»ãã®ã¸ã§ããä½æãã

https://docs.gitlab.com/ee/ci/yaml/README.html#onlyexcept-basic

`only` ã `except` ã¨ããã­ã¼ã¯ã¼ããå®ç¾©ããã¨ãããæ¡ä»¶ã®å ´åã®ã¿ã¸ã§ããå®è¡ããããããæ¡ä»¶ãé¤ãå ´åã®ã¿ã¸ã§ããå®è¡ããããããã¨ãã§ãã¾ãã

ä¾ãã°ãã¿ã°ãããã·ã¥ãããæã ãå®è¡ããããã«å®ç¾©ãããã

```yaml
only:
  - tags
```

ããç¹å®ã®ãã©ã³ããããã·ã¥ãããæã¯å®è¡ãããªãããã«å®ç¾©ããããããã¨ãã§ãã¾ãã

```yaml
except:
  - master
```

ã¾ããæ­£è¦è¡¨ç¾ã«ãå¯¾å¿ãã¦ããã®ã§ç¹å®ã®ãã¬ãã£ãã¯ã¹ãã¤ãããã©ã³ããããã·ã¥ãããæã ãå®è¡ããã¨ãããããªå®ç¾©ãå¯è½ã§ãã

```yaml
only:
  - /^release-.*$/
```

## 5. when:manualï¼å®å¿å®å¨ãªããã­ã¤

https://docs.gitlab.com/ee/ci/yaml/README.html#whenmanual

`when:manual` ãã¸ã§ãã«å®ç¾©ããã¨ãæåã§å®è¡ãã¿ã³ãã¯ãªãã¯ãããã¸ã§ããå®è¡ãããããã«ãªãã¾ãã

<figure class="figure-image figure-image-fotolife" title="ã¸ã§ãã«when:manualãå®ç¾©ããã¨ãã®ãã¤ãã©ã¤ã³">[f:id:gyuuuutan:20181202012650p:plain]<figcaption>ã¸ã§ãã«when:manualãå®ç¾©ããã¨ãã®ãã¤ãã©ã¤ã³</figcaption></figure>

ä¸è¨ã¯ `when:manual` ãããã­ã¤ã¸ã§ãã«å®ç¾©ããéã®ãã¤ãã©ã¤ã³ã§ãããããã·ã¥ããã ãã§ã¯ããã­ã¤ã¸ã§ãã¯å®è¡ããã¾ããã  
ãâ¶ï¸ãã®å®è¡ãã¿ã³ãæ¼ããã¨ã§åãã¦ããã­ã¤ã¸ã§ããå®è¡ããã¾ãã

ããã¯æ¬çªç°å¢ã¸ã®ããã­ã¤ã¸ã§ãã«æç¨ã§ãä¸æ®ã®äºæãé²ããã¨ãã§ãã¾ãã

## 6. artifactsï¼ã«ãã¬ãã¸ã¬ãã¼ããå¯è¦åãã

https://docs.gitlab.com/ee/user/project/pipelines/job_artifacts.html

`artifacts` ã­ã¼ã¯ã¼ãã¯ã¸ã§ãã§çæãããææç©ããã¦ã³ã­ã¼ãå¯è½ã«ããããGitLab Pages ã§å¬éãããããã¨ãã«æç¨ã§ãã

ä¾ãã°ãä¸è¨ã®ããã«å®ç¾©ããã¨ãã¦ããããã¹ãã§çæãããã«ãã¬ãã¸ã¬ãã¼ãããã¤ãã©ã¤ã³ãããã¦ã³ã­ã¼ãã§ããããã«ãªãã¾ãã

```yaml
script:
  - pytest --cov-report=html
artifacts:
  paths:
    - htmlcov
```

<figure class="figure-image figure-image-fotolife" title="ã¸ã§ãã«artifactsãå®ç¾©ããã¨ãã®ãã¤ãã©ã¤ã³">[f:id:gyuuuutan:20181202012834p:plain]<figcaption>ã¸ã§ãã«artifactsãå®ç¾©ããã¨ãã®ãã¤ãã©ã¤ã³</figcaption></figure>

**ãè£è¶³ã**  
ã«ãã¬ãã¸ã¬ãã¼ãã¯ GitLab Pages ã®æ©è½ãå©ç¨ãããã¨ã§çä¾¡ãçºæ®ãã¾ãã

GitLab Pages ã¯ GitHub Pages ã¨åããããªæ©è½ã§ããã GitLab ã®ãªãã¸ããªããéçãµã¤ããå¬éããããã®æ©è½ã§ãã

ç¡æã§å©ç¨ã§ããããhttps ã«å¯¾å¿ãã¦ãããã¨ãã£ãç¹å¾´ãããã¾ãã

https://docs.gitlab.com/ce/user/project/pages/

GitLab Pages ã¯ä»¥ä¸ã® 3 ã¹ããããè¡ãã ãã§éçãµã¤ããèªåã§çæãã¦ããã¾ãã

1. `public` ã¨ããååã®ãã£ã¬ã¯ããªã«éçãã¡ã¤ã«ãéç½®ããã
2. `.gitlab-ci.yml` ã«ããã­ã¤ã®ã¸ã§ããå®ç¾©ãã
3. å®ç¾©ããã¸ã§ããå®è¡ãã

ä¸è¨ã® URL ã¯ä¸è¨ã® 3 ã¹ãããã§èªåçæãããå®éã® URL ã§ãã  
ã«ãã¬ãã¸ã¬ãã¼ããå±éãã¦ãã¾ãã

https://jumpyoshim.gitlab.io/django-polls

çæãããã«ãã¬ãã¸ã¬ãã¼ãã `public` ãã£ã¬ã¯ããªã«éç½®ã `artifacts` ã® `paths` ã«å®ç¾©ããã ãã§ãã

ããã§æ³¨æããªããã°ãªããªãã®ã `deploy` ã¹ãã¼ã¸ã§ãªãã¨ GitLab Pages ã®ãã¡ã¤ã³ãçæãã¦ãããªããã¨ã§ããï¼ã©ããã¦ããã¾ããããªãã¦æ°æéæ©ã¾ããã¾ãããï¼

```yaml
pages:
  stage: deploy
  image: alpine:latest
  script:
    - mv htmlcov/ public/
  artifacts:
    paths:
      - public
  only:
    - master
```

FYI: https://gitlab.com/help/user/project/pages/index.md

# åèä¾

ä¸è¨ã® Tips ãæ´»ç¨ãã `.gitlab-ci.yml` ã®ä¾ã§ãã
ä»¥ä¸ãå®è¡ãã¦ãã¾ãã

- GitLab ã³ã³ããã¬ã¸ã¹ããªã¸ã® Docker ã¤ã¡ã¼ã¸ã®ããã·ã¥
- ã¦ããããã¹ã
- ææ³ãã§ãã¯
- ã³ã¼ãã¡ããªã¯ã¹è¨æ¸¬
- Heroku ã¸ã®ããã­ã¤
- GitLab Pages ã®å¬éè¨­å®

FYI: https://gitlab.com/jumpyoshim/django-polls/blob/master/.gitlab-ci.yml

```yaml
.build_template: &build_definition
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY

build_for_master:
  <<: *build_definition
  script:
    - docker build --pull -t $CI_REGISTRY_IMAGE:latest .
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - master

build_for_tags:
  <<: *build_definition
  script:
    - docker build --pull -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
  only:
    - tags

.test_template: &test_definition
  stage: test
  image: python:3.7
  before_script:
    - pip install pipenv
    - pipenv install --system --dev
  except:
    - master

pytest:
  <<: *test_definition
  services:
    - postgres:latest
  variables:
    POSTGRES_DB: custom_db
    POSTGRES_USER: custom_user
    POSTGRES_PASSWORD: custom_password
  script:
    - export DATABASE_URL=postgres://$POSTGRES_USER:$POSTGRES_PASSWORD@postgres:5432/$POSTGRES_DB
    - python3 manage.py collectstatic --noinput
    - pytest --cov-report=html --tb=line
  after_script:
    - mv htmlcov/ public/
  artifacts:
    paths:
      - public

flake8:
  <<: *test_definition
  script:
    - flake8

radon:
  <<: *test_definition
  script:
    - radon cc -n C .
    - radon mi -n B .

deploy:
  stage: deploy
  image: ruby:2.5
  script:
    - gem install dpl
    - dpl --provider=heroku --app=$HEROKU_APP --api-key=$HEROKU_API_KEY --skip-cleanup
  only:
    - tags
  when: manual

pages:
  stage: deploy
  image: alpine:latest
  script:
    - echo 'Upload the coverage report'
  artifacts:
    paths:
      - public
  only:
    - master
```

# ãããã«

**ä¿ºç**ã¨æ¸ãã¦ããã¾ããããããã®ãã­ã¥ã¡ã³ããè¨äºãè·å ´ã®ã¨ã³ã¸ãã¢ãGitLab ã³ãã¥ããã£ã®æ¹ã®å©ãã§å¾ããã¨ãã§ãã Tips ã§ãããã®å ´ãåãã¦ãç¤¼ç³ãä¸ãã¾ãã  
OSS ã¸ã®ã³ã³ããªãã¥ã¼ããªã©ãä½ãããã®å½¢ã§æ©è¿ãããã¦ããããã¨æã£ã¦ãã¾ãã

**ãã­ã¥ã¡ã³ã**

- [Configuration of your jobs with .gitlab-ci.yml](https://docs.gitlab.com/ce/ci/yaml/)
- [GitLab Pages](https://gitlab.com/help/user/project/pages/index.md)

**è¨äº**

- [.gitlab-ci.yml ã«ããã¸ã§ãã®è¨­å®æ¹æ³(æ¥æ¬èªè¨³)](https://qiita.com/ynott/items/1ff698868ef85e50f5a1)

**GitLab ã³ãã¥ããã£**

- [Slack](https://gitlab-jp.herokuapp.com/)
- [Connpass](https://gitlab-jp.connpass.com/)

GitLab ã«é¢ããæ©è½ãè±ç¥è­ã 7 ã¤å³é¸ãã¦ã¿ã¾ããã  
ãããããããã°ã®ããã¦ã¿ã¦ãã ããã

https://qiita.com/saka2jp/items/d5a63bdd3681843866f8
