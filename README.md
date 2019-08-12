# 基礎 pip package 開發範例以及環境建置流程

這份 Project  是從閱讀[本篇文章][good-practice]以及基礎 [PyTest 文件][pytest-test-assert-example]衍生而來一個簡單的 pip package 範例, 這個範例的主要目的是展示透過基礎的 python 工具建置一整個 [pip package][pip-packing] 的流程. 

寫這份文件的目的也是為了展示一個相對完整的文件應該有的樣貌, 有 SOP 以及足夠的解釋項目. 所以本文所有步驟都是經過驗證, 以確保下列步驟在本文的列出環境下是可以運作的. 如果讀者本身環境已經建置相關工具並可以正常使用, 可以忽略第一項基礎環境建置的步驟.

# 安裝環境

* 主機: Intel® Core™ i7-6700HQ CPU @ 2.60GHz × 4 , 8 GB 記憶體
* 作業系統: Ubuntu 18.04.3 LTS

# 步驟

一般的 python 開發者為了不影響原生作業系統環境的運作, 都會約定成俗的建置一個虛擬環境來開發, 所以在此我會把整個步驟分為下列幾個項目.

1. 基礎環境建置
2. 建立虛擬環境
3. 建置基礎套件環境
4. 建置基礎開發流程

# 1. 基礎環境建置

## 所需套件

* `python3`: python 3 直譯器
* `pip3`: python 3 套件管理系統
* `virtualenv`: python 虛擬環境建置系統

以上所有指令都可以透過 `--version` 參數確認是否已經安裝, 如果沒有的話再參考下列步驟安裝, 已安裝的指令過則無須再安裝.

## 安裝 python 3

請透過 `apt`  先更新套件版本並安裝 python 3, `-y` 參數系統就會直接安裝並且不會再跟使用者確認.

```sh
sudo apt update
sudo apt install -y python3
```

## 安裝 pip3

在建置相關環境之前, 我們必須先安裝 `pip3` 才能安裝 `virtualenv` 以及相關所需的套件.

執行下列指令安裝 `pip3`.

```sh
$ sudo apt install -y python3-pip
```

## 安裝 virtualenv

採用  `virtualenv` 虛擬環境最大的好處是它可以依照需求建立獨立的 python 2 或是 python 3 執行環境. 之後我們會用到相關參數建立 python 3 執行環境而不會跟系統內定的 python 2 衝突. 

執行下列指令安裝 `virtualenv`,

```sh
$ sudo apt install -y virtualenv
```

或是使用 `pip` 安裝.

```sh
$ sudo pip3 install virtualenv
```

通常為了保險起見像系統套件, 我會先用 `apt` 套件, 再考慮 `pip3` 套件. 畢竟 Ubuntu 維護 package 的方式與正統的 python 社群不太一樣. 如果 Ubuntu 系統太舊, 就得個案處理 `pip3` 與 `apt`套件相容性問題.


# 2. 建立虛擬環境

虛擬環境的路徑可以依照個人需求設定, 在此我們就設定在 Project 目錄底下的 `runenv` 目錄.

假設我們在 Project 目錄下. 請輸入下列指令建立環境, 

```sh
$ virtualenv --python=python3 runenv
```

`--python` 參數是指定 python 版本. `virtualenv` 會自行搜尋系統路徑內已安裝的 python 版本. 舉例我們可以用 `--python=python3.6` 指定安裝的環境為 python 3.6. 如果不確定就維持 ` --python=python3` 即可.

之後我們輸入下列指令切到剛剛建立的環境, 測試是否可以正常運作,

```sh
$ source runenv/bin/activate
```

如果在提示符號開頭出現`(runenv)`, 代表我們目前運行在剛剛建立的 python 3 環境裡.

接下來輸入下列指令離開目前的環境,

```sh
(runenv)$ deactivate
```

這時候就可以看到符號開頭的`(runenv)`已經消失 , 代表我們已經離開剛剛的環境. 本文中之後所有python 3的動作, 包括安裝 `pip3` 套件, 都會假設我們已經先切換到 `(runenv)`虛擬環境當中.

# 3. 建置基礎套件環境

## 使用 setuptools 包裝套件

在 python 裡常常會遇到路徑問題, 例如自己寫的 python codes 在 `import` 的時候常常遇到 package 找不到. 所以比較妥善的方式是把自己的程式碼包成一個 package. Python 有很強大的管理工具集, 但是也是大亂鬥居多, 所以我們會列出 `pip3` 會用到以及支援的工具.

- `setuptools`: 目前最多人用的套件管理工具集, `pip3` 有很多功能骨子裡還是 `setuptools`
- `wheel`:  package 打包工具, 打包出來的 `xxx.whl` 檔骨子裡就是 `zip` 檔

## 安裝相關工具

通常 `virtualenv` 都會建置這些基礎的套件, 不過為了保險我們先檢查以上工具是否存在.

檢查套件,

```sh
(runenv)$ pip3 list | grep 'setuptools'
(runenv)$ pip3 list | grep 'wheel'
```

如果沒有看到還是可以透過下列指令安裝相關套件,

```sh
(runenv)$ pip3 install setuptools wheel
```

## 設定 setup.py

根據之前閱讀的[文章](good-practice), 我們只需要確認根目錄底下有 `setup.py`, 而且`setup.py`裡有起碼底下的程式碼就可以讓 `pip` 就可以管理我們的套件,

```python
from setuptools import setup, find_packages

setup(name="PACKAGENAME",  packages=find_packages())
```

其中 `PACKAGENAME` 就是之後 `pip install` 所使用的套件名稱, 如果想要知道的參數, 請詳閱[官方文件][pip-packing].

# 4. 建置基礎開發流程

接下來就進入到本篇的主題, 最基礎的 python package 建置流程. 

## python packaging 的一些基礎知識

首先我建議大家先讀過[這份文件][python-packages-basic]先大概了解 python modules 的基本運作方法. Python module 基本上都是以目錄的形式居多, 目錄下的子目錄都視為 sub module. 讀取 submodule 的方法則是以 module 與 sub module 之間加上 `.` 為分隔. Eg: 目錄 `a/b` 的 package 為 `a.b`. Python 在搜尋 packages 的時候, 都會以當前目錄的 `___init___.py` 為進入點, 如果有需要初始化的程式碼可以放在這個檔案裡. 如果是在目錄底下的其他檔案, 則視為其他模組. Eg: `a/c.py` 視為 `a.c`.

## 目錄結構

根據之前閱讀的[文章][good-practice]之中有詳細說明各種不同的目錄套件結構以及相關的使用情境. 我們設定一個比較簡單的情況來整理我們的程式碼.

假設我們只需要一個套件, 但是我們不想把測試碼包進我們的套件, 此時可以用下列目錄結構.

```sh
├── my_package
│   ├── __init__.py
│   └── power.py
├── README.md
├── setup.py
└── tests
    └── test_power.py
```

因為 `pip` 只會把有 `__init__.py` 檔案的目錄加入 package list, 所以只有 `my_package` 裡的檔案會被加入. 至於 `tests` 目錄是放置測試程式碼的部份, 至於如何使用測試框架, 之後會敘述.

## 統整 import 路徑

在這個範例當中, `my_package` 有下列結構,

```sh
./my_package
./my_package/__init__.py
./my_package/power.py
```
其中 `power_2` 函式是放在 `__init__.py` 當中, 所以 `import my_package` 很直覺得就可以使用 `my_package.power_2` 函式, 但是如果是要使用 `power_3` 函式得用 `import my_package.power` 並呼叫 `my_package.power.power_3`, 相較之下就沒這麼直覺了.  所以最好的方式是在 `__init__.py` 裡就 `import` 所有需要的函式就可以直覺的使用 `my_package.power_3`了. 有篇[短文章][exporting-functions-from-init]簡短的敘述了相關狀況.

`__init__.py`範例

```python
from .power import *
```

## 測試

在現代的軟體開發流程, 最講究的莫過於 DevOp. 以至少每天為單位自動化建置, 測試, 打包, 發行或是部屬環境.  為了確保軟體品質, 最重要的一環還是測試. 有正確的測試才能確保任何修正不會產生 side effect.

Python 有非常多的測試框架, 但是目前同時可以做單元測試以及 BDD (Behavior Driven Development) 的框架比較多人用的選擇是 `pytest`, 在此我們也是採用 `pytest` 作為我們範例中的測試框架.

### 安裝 pytest

首先執行下列指令確認 `pytest` 是否已經安裝,

```sh
(runenv)$ pip3 list | grep 'pytest'
```

如果並未安裝則執行下列指令,

```sh
(runenv)$ pip3 install pytest
```
### 設定測試開發環境

根據之前閱讀的[文章][good-practice], 我們可以透過下列指令把目前的 Project 目錄當成 package 目錄, 

```sh
(runenv)$ pip3 install -e .
```

其中參數 `-e` 代表可編輯, 意思就是我們只是在當下的環境建立一個連結到 python 的 package 路徑之中, 這樣就算 package 程式碼有修改, 還是會立即生效. 在執行 `pytest` 之前必須確保這個動作有執行, 不然測試會出現錯誤.

### 寫測試

`pytest` 預設會搜尋整個 Project 目錄任何帶有 `test` 字眼的 `python` 檔案, 並把裡面帶有任何 `test` 字眼的函式當作測項, 只要沒有任何 exception 發生就算通過測項.

所以在測試項目當中, 我們會以下列的風格來寫測試,

```python
def test_xxx():
    assert .....
```

如範例中 `test_power.py`,

```python
def test_power_2():
    assert my_package.power_2(2) == 4
```

此測項是測試 `my_package.power_2` 的功能是計算參數的平方函式, 我們預期此函式帶參數 2 的結果為 4. 如果這個函式沒有滿足這項測試則會出現 Failed. 想要知道更多可以詳閱 `pytest` 的[官方文件][pytest-test-assert-example]. 

## 執行測試

執行測試,

```sh
(runenv)$ pytest
```

如果所有測試都通過螢幕資就會顯示綠色的文字, 不然就會出現紅色文字.

## 打包

以下內容[從此處][pip-packing]節錄.

`setuptools` 打包指令,

```sh
python3 setup.py sdist bdist_wheel
```

其中參數 `sdist` 為打包原始程式碼, `bdist_wheel` 只有打包 `wheel` 格式的 binary  檔案.

打包的結果會放在 `dist` 目錄當中, 結果如下,

```sh
dist
├── [-rw-r--r--]  pip_simple_demo-0.0.0-py3-none-any.whl
└── [-rw-r--r--]  pip-simple-demo-0.0.0.tar.gz

```

[exporting-functions-from-init]: https://gist.github.com/CMCDragonkai/510ce9456a0429f616baa243d1de3dbf "Exporting Modules and Functions from Python __init__.py"
[python-packages-basic]: https://docs.python.org/3/tutorial/modules.html#packages "6.4. Packages"
[pip-packing]: https://packaging.python.org/tutorials/packaging-projects/ "Packaging Python Projects"

[pytest-test-assert-example]: http://doc.pytest.org/en/latest/assert.html#assert "The writing and reporting of assertions in tests"
[good-practice]: http://doc.pytest.org/en/latest/goodpractices.html "Good Integration Practices"
