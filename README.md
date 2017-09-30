# docs-sphinx_summary
## 목표
1. pdf 관련 : 오픈스택 문서가 openstack-manuals 있던 것이 문서팀 활동이 줄어들면서 설치 가이드가 앞으로는 manuals는 필수적인것만 종합하고, nova, netron 설치는 각 저장소에 저장

2. conf.py : pdf 지원하도록 바꾸어야 하는데 기존에서 어떻게 바꾸어야 한다 라는 아이디어를 가지고 있어야 일괄적으로 바꿀 수 있다. 일단은 수동으로 경로 찍고, 나중에 차차 자동화 하는 방식으로.

3. requirements.txt test-requirements.txt / test-requirements.txt -> doc-tools, docs-theme 두개가 필요함. doc-tools 없는 것이 있을 것으로 예상. 이 부분 추가해야함.

## conf.py의 폰트 경로 지정
'''
latex_elements = {
    # The paper size ('letterpaper' or 'a4paper').
    # 'papersize': 'letterpaper',

    # set font (TODO: different fonts for translated PDF document builds)
    'fontenc': '\\usepackage{fontspec}',
    'fontpkg': '''\
\defaultfontfeatures{Scale=MatchLowercase}
\setmainfont{Liberation Serif}
\setsansfont{Liberation Sans}
\setmonofont[SmallCapsFont={Liberation Mono}]{Liberation Mono}
''',

    # The font size ('10pt', '11pt' or '12pt').
    # 'pointsize': '10pt',

    # Additional stuff for the LaTeX preamble.
    # 'preamble': '',
}
'''

* https://tex.stackexchange.com/questions/103704/how-to-properly-install-and-use-a-new-font-with-lualatex
'''
pdfdoctheme에 직접 해보지는 못하고 Makefile이 있는 doc 경로에 
\setmainfont[
    Path           = /home/jang/.local/share/fonts/,
    Extension      = .ttf,
    Ligatures      = TeX,
    BoldFont       = Hack-Bold,
    ItalicFont     = Hack-Italic,
    BoldItalicFont = Hack-BoldItalic
]{Hack-Regular}
같이 수정해서 make를 해보니 오류없이 잘 생성이 되었습니다.
이렇게 되면 실제 openstack-manuals가 설치되는 서버의 디렉터리 경로와, 폰트를 모아놓을 디렉토리 결정, 각 언어별 폰트 테스트 후 테마 수정 을 앞으로 수행하면 될듯 합니다.
'''

## openstack-manuals에서 기본적인 rst -> pdf 변환 과정
### build_rsh.sh PDF generation part
* http://git.openstack.org/cgit/openstack/openstack-manuals/tree/tools/build-rst.sh#n96
'''
    # PDF generation
    if [ "$PDF" = "1" ] ; then
        set -x
        sphinx-build -E -W -d $DOCTREES -b latex \
            $TAG_OPT $DIRECTORY/source $BUILD_DIR_PDF
        make -C $BUILD_DIR_PDF
        cp $BUILD_DIR_PDF/*.pdf $BUILD_DIR/
        set +x
    fi
'''

### OpenStack murano setup.cfg example
* https://github.com/openstack/murano/blob/master/setup.cfg#L84
'''
[build_sphinx]
all_files = 1
build-dir = doc/build
source-dir = doc/source
warning-is-error = 1
'''

### html에서 pdf로 빌드
* http://git.openstack.org/cgit/openstack/openstack-manuals/tree/tox.ini#n41
'''
[testenv:pdfs]
commands =
  {toxinidir}/tools/build-all-rst.sh --pdf
'''

###궁금한 사항
* setup.cfg와 build_rsh.sh에서 $DIRECTORY / $BUILD_DIR 어떠한 상관?

----------

## Setuptools integration
* http://www.sphinx-doc.org/en/stable/setuptools.html
* 목적 : 통합 목적?

### Steuptools를 이해하기 위한 배경지식
#### 파이썬 프로젝트 시작하기 - Virtualenv
* http://www.flowdas.com/blog/%ED%8C%8C%EC%9D%B4%EC%8D%AC-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-virtualenv/

#### 파이썬 프로젝트 시작하기 - Distutils
* http://www.flowdas.com/blog/%ED%8C%8C%EC%9D%B4%EC%8D%AC-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-distutils/
* 목적 : installing third-party modules. -> write a setup script (setup.py by convention)

#### 파이썬 프로젝트 시작하기 - Setuptools
* http://www.flowdas.com/blog/%ED%8C%8C%EC%9D%B4%EC%8D%AC-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-setuptools/
* 목적 : Building and Distributing Packages

## Distributing sphinx generated documentation with distutils
### How the setup.py file should look like
* https://groups.google.com/forum/#!topic/sphinx-users/VmodfxFz4LM
'''
If you are using setuptools or distribute, you can do the following to
include the documentation in the source distribution (untested):

Create "setup.cfg" with the following content:

[aliases]
build_html = build_sphinx -b html --build-dir build/sphinx/html
build_pdf = build_sphinx -b pdf --build-dir build/sphinx/pdf
sdist = build_html build_pdf sdist

This tells "setuptools" to build both HTML and PDF documentation with
sphinx before, whenever you create a source distribution.  Then add
the following lines to "MANIFEST.in" to tell setuptools to actually
include the built documentation into the source distribution:

graft build/sphinx/html
graft build/sphinx/pdf

Now "python setup.py sdist" should give you an archive which includes
the documentation both in HTML and in PDF format.  But don't do this,
if your package is distributed on the package index, because in this
case it is just wasted space and traffic.  Most users use easy_install
or pip to install packages from the package index, without ever
actually seeing the source distribution itself.  Better upload the
documentation on packages.python.org, so that users can read it online
(and optionally download a PDF file).  Of course, this does not apply,
if you distribute your software in other ways.
'''

## 아직 분석 못하거나 모르는 것들
* http://git.openstack.org/cgit/openstack-infra/project-config/tree/zuul/mapping.yaml#n162
* 안 나옴 : http://git.openstack.org/cgit/openstack-infra/openstack-zuul-jobs/tree/zuul.yaml#n46