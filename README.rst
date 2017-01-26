doxypypy
========

*Pythonのバージョンのdoxypy、Python用のDoxygenフィルタ。

まず　どんなものか？
------

今のところ、DoxygenはPythonのサポートを制限しています。
これはPythonのコメントを認識しますが、
そうでなければその言語をJavaとほぼ同じように扱います。これは、
docstrings、キーワード引数、ジェネレータ、ネストされた関数、
デコレータ、またはラムダ式のような基本的なPython構文構造を理解していません。
同様に、doctestやZOPEスタイルのインタフェースのような従来の構造を理解していません。
ただし、
入力ソースコードを予想どおりにするために使用できるインラインフィルタをサポートしています。

優れたdoxypyは、DoxygenのコマンドをPythonのdocstringに埋め込み、
Doxygenの通常の入力フィルタ処理ごとにDoxygenで認識されたコメントに変換することができます。
しかしそれは、以前に言及した困難な分野のいずれにも対処していない。

このプロジェクトは、ドキンピーのフォークとして始まりましたが、すぐにはっきりと区別されました。
この時点では、同じコードをほとんど共有していません（ただし、その場合は元のライセンスを保持しています）。
これはdoxypyと同じコマンドラインオプションをすべてサポートしています
docstrings以外のPython構文も処理できます。

サポートされている追加構文
---------------------------

Pythonは、関数とクラスの両方で関数とクラスを持つことができます。 Doxygenは、この概念をネームスペースという概念で最もよく理解しています。したがって、このフィルタは、すべての関数とクラスに名前空間をマークするDoxygenタグを提供できます。これは、Doxygenの内部関数のドキュメントを親のドキュメンテーションとマージする問題を扱っています。

名前が二重アンダースコアで始まるPythonクラスのメンバーは、その言語に応じて変更され、プライベートに保たれます。 Doxygenはこれをネイティブではまだ理解していないので、このフィルタはDoxygenタグを追加してプライベートな変数にラベル付けします。

Pythonは頻繁にdocstrings内にdoctestを埋め込みます。このフィルタは、ドキュメンテーション文字列のそのようなセクションをマークしてコードとして表示することを自明にします。

ZOPEスタイルのインターフェイスはクラス定義をインターフェイス定義にオーバーロードし、埋め込み変数の割り当てを使用して属性を識別し、特定の関数呼び出しを使用してインターフェイスの順守を示します。さらに、彼らは頻繁にdocstrings以外のコードを持っていないので、docstringを単純に削除するとPythonが壊れてしまいます。このフィルタはこれらのインタフェースの基本的な理解を持ち、必要に応じてDoxygenタグを供給してそれに応じて処理します。

基本的には、Pythonのドキュメントストリングは、人間ではなく機械であり、従来の構造化テキストを超えて特別なマークアップを行うべきではありません。このフィルタはヒューリスティックにPythonのドキュメントストリングを調べ、PEP 257の複合サンプルや一般的に厳しいGoogle Pythonスタイルガイドに従ったものは、適切なDoxygenタグが自動的に追加されます。

使い方
-----
このプロジェクトはdoxypyとは根本的に異なるアプローチをとっています。ステートマシンに結合された正規表現を使用して構文を把握するのではなく、Python独自の抽象構文ツリーモジュールを使用して興味のある項目を抽出します。 autobriefオプションが有効な場合、docstringは正規表現のセットとコルーチンのプロデューサ/コンシューマのペアを介して解析されます。

例
--
Example
-------

このフィルタは、次のようなコードを正しく処理します（ただし
考案された）例：

.. code-block:: python

    def myfunction(arg1, arg2, kwarg='whatever.'):
        """
        Does nothing more than demonstrate syntax.

        This is an example of how a Pythonic human-readable docstring can
        get parsed by doxypypy and marked up with Doxygen commands as a
        regular input filter to Doxygen.

        Args:
            arg1:   A positional argument.
            arg2:   Another positional argument.

        Kwargs:
            kwarg:  A keyword argument.

        Returns:
            A string holding the result.

        Raises:
            ZeroDivisionError, AssertionError, & ValueError.

        Examples:
            >>> myfunction(2, 3)
            '5 - 0, whatever.'
            >>> myfunction(5, 0, 'oops.')
            Traceback (most recent call last):
                ...
            ZeroDivisionError: integer division or modulo by zero
            >>> myfunction(4, 1, 'got it.')
            '5 - 4, got it.'
            >>> myfunction(23.5, 23, 'oh well.')
            Traceback (most recent call last):
                ...
            AssertionError
            >>> myfunction(5, 50, 'too big.')
            Traceback (most recent call last):
                ...
            ValueError
        """
        assert isinstance(arg1, int)
        if arg2 > 23:
            raise ValueError
        return '{0} - {1}, {2}'.format(arg1 + arg2, arg1 / arg2, kwarg)

注意すべき点がいくつかあります。:

1.特別なタグは使用されません。人間が読めるセクションヘッダーのベストプラクティス
十分です。

2.いくらかの柔軟性は許されます。セクションの最も一般的な名前は受け入れられますが、
項目と説明はコロンまたはダッシュで区切ることができます。

3.ブリーフは最初の項目でなければならず、1行以上でなければなりません。

4.サンプルセクションにスローされたものはすべてコードとして扱われるので、
doctestsのための完璧な場所。

追加の包括的な例がテスト領域にあります。

doxypypyのインストール
-------------------

インストールにはcode： `pip`または：code：` easy_install`のいずれかを使用できます。
どちらかを実行する：

.. code-block:: shell

    pip install doxypypy

または：

.. code-block:: shell

    easy_install doxypypy

管理者の特権でこのトリックを行う必要があります。

doxypypy出力のプレビュー
--------------------------
インストールに成功すると、doxypypyはコマンドラインから
フィルタリングされた結果をプレビューするには：

.. code-block:: shell

    doxypypy -a -c file.py

通常は、出力をテキストエディタで表示するためにファイルにリダイレクトする必要があります。

.. code-block:: shell

    doxypypy -a -c file.py > file.py.out


Doxygenからのdoxypypyの呼び出し
------------------------------

DoxygenをdoxypypyでPythonコードを実行させるには、FILTER \ _PATTERNSを設定します
タグをDoxyfileに追加します。

.. code-block:: shell

    FILTER_PATTERNS        = *.py=py_filter

`py_filter`はあなたのパスでシェルスクリプト（またはWindowsバッチ）として利用可能でなければなりません
ファイル）。特定のディレクトリで `py_filter`を実行したい場合は、
完全パスまたは相対パス。

Unixライクなオペレーティングシステムの場合、 `py_filter`は次のようになります：

.. code-block:: shell

    #!/bin/bash
    doxypypy -a -c $1
    
    
Windowsでは、バッチファイルは `py_filter.bat`という名前でなければなりません。
1行を含む：

.. code-block:: shell

    doxypypy -a -c %1
    
    
いつものようにDoxygenを実行すると、すべてのPythonコードがdoxypypyで実行されるはずです。 Be
Doxygenの出力を最初に慎重に閲覧してください
Doxygenは適切に見つけられ、doxypypyを実行しました。

    
.. _Doxygen: http://www.stack.nl/~dimitri/doxygen/
.. _doxypy: https://github.com/Feneric/doxypy
.. _PEP 257: http://www.python.org/dev/peps/pep-0257/
.. _Google Python Style Guide: http://google-styleguide.googlecode.com/svn/trunk/pyguide.html?showone=Comments#Comments


    
================================================================================================================================
====================================================doxypypy  eng  =============================================================
================================================================================================================================


doxypypy
========

*A more Pythonic version of doxypy, a Doxygen filter for Python.*

Intent
------

For now Doxygen_ has limited support for Python.  It recognizes Python comments,
but otherwise treats the language as being more or less like Java.  It doesn't
understand basic Python syntax constructs like docstrings, keyword arguments,
generators, nested functions, decorators, or lambda expressions.  It likewise
doesn't understand conventional constructs like doctests or ZOPE-style
interfaces.  It does however support inline filters that can be used to make
input source code a little more like what it's expecting.

The excellent doxypy_ makes it possible to embed Doxygen commands in Python
docstrings, and have those docstrings converted to Doxygen-recognized comments
on the fly per Doxygen's regular input filtering process.  It however does not
address any of the other previously mentioned areas of difficulty.

This project started off as a fork of doxypy but quickly became quite distinct.
It shares little (if any) of the same code at this point (but maintains the
original license just in case).  It is meant to support all the same command
line options as doxypy, but handle additional Python syntax beyond docstrings.

Additional Syntax Supported
---------------------------

Python can have functions and classes within both functions and classes.
Doxygen best understands this concept via its notion of namespaces.  This filter
thus can supply Doxygen tags marking namespaces on every function and class.
This addresses the issue of Doxygen merging inner functions' documentation with
the documentation of the parent.

Python class members whose names begin with a double-underscore are mangled
and kept private by the language.  Doxygen does not understand this natively
yet, so this filter additionally provides Doxygen tags to label such variables
as private.

Python frequently embeds doctests within docstrings.  This filter makes it
trivial to mark off such sections of the docstring so they get displayed as
code.

ZOPE-style interfaces overload class definitions to be interface definitions,
use embedded variable assignments to identify attributes, and use specific
function calls to indicate interface adherence.  Furthermore, they frequently
don't have any code beyond their docstrings, so naively removing docstrings
would result in broken Python.  This filter has basic understanding of these
interfaces and treats them accordingly, supplying Doxygen tags as appropriate.

Fundamentally Python docstrings are meant for humans and not machines, and ought
not to have special mark-up beyond conventional structured text.  This filter
heuristically examines Python docstrings, and ones like the sample for complex
in `PEP 257`_ or that generally follow the stricter `Google Python Style Guide`_
will get appropriate Doxygen tags automatically added.

How It Works
------------

This project takes a radically different approach than doxypy.  Rather than use
regular expressions tied to a state machine to figure out syntax, Python's own
Abstract Syntax Tree module is used to extract items of interest.  If the
`autobrief` option is enabled, docstrings are parsed via a set of regular
expressions and a producer / consumer pair of coroutines.

Example
-------

This filter will correctly process code like the following working (albeit
contrived) example:

.. code-block:: python

    def myfunction(arg1, arg2, kwarg='whatever.'):
        """
        Does nothing more than demonstrate syntax.

        This is an example of how a Pythonic human-readable docstring can
        get parsed by doxypypy and marked up with Doxygen commands as a
        regular input filter to Doxygen.

        Args:
            arg1:   A positional argument.
            arg2:   Another positional argument.

        Kwargs:
            kwarg:  A keyword argument.

        Returns:
            A string holding the result.

        Raises:
            ZeroDivisionError, AssertionError, & ValueError.

        Examples:
            >>> myfunction(2, 3)
            '5 - 0, whatever.'
            >>> myfunction(5, 0, 'oops.')
            Traceback (most recent call last):
                ...
            ZeroDivisionError: integer division or modulo by zero
            >>> myfunction(4, 1, 'got it.')
            '5 - 4, got it.'
            >>> myfunction(23.5, 23, 'oh well.')
            Traceback (most recent call last):
                ...
            AssertionError
            >>> myfunction(5, 50, 'too big.')
            Traceback (most recent call last):
                ...
            ValueError
        """
        assert isinstance(arg1, int)
        if arg2 > 23:
            raise ValueError
        return '{0} - {1}, {2}'.format(arg1 + arg2, arg1 / arg2, kwarg)

There are a few points to note:

1.  No special tags are used.  Best practice human-readable section headers
are enough.

2.  Some flexibility is allowed.  Most common names for sections are accepted,
and items and descriptions may be separated by either colons or dashes.

3.  The brief must be the first item and be no longer than one line.

4.  Everything thrown into an examples section will be treated as code, so it's
the perfect place for doctests.

Additional more comprehensive examples can be found in the test area.

Installing doxypypy
-------------------

One can use either :code:`pip` or :code:`easy_install` for installation.
Running either:

.. code-block:: shell

    pip install doxypypy

or:

.. code-block:: shell

    easy_install doxypypy

with administrator privileges should do the trick.

Previewing doxypypy Output
--------------------------

After successful installation, doxypypy can be run from the command line to
preview the filtered results with:

.. code-block:: shell

    doxypypy -a -c file.py

Typically you'll want to redirect output to a file for viewing in a text editor:

.. code-block:: shell

    doxypypy -a -c file.py > file.py.out

Invoking doxypypy from Doxygen
------------------------------

To make Doxygen run your Python code through doxypypy, set the FILTER\_PATTERNS
tag in your Doxyfile as follows:

.. code-block:: shell

    FILTER_PATTERNS        = *.py=py_filter

`py_filter` must be available in your path as a shell script (or Windows batch
file).  If you wish to run `py_filter` in a particular directory you can include
the full or relative path.

For Unix-like operating systems, `py_filter` should like something like this:

.. code-block:: shell

    #!/bin/bash
    doxypypy -a -c $1

In Windows, the batch file should be named `py_filter.bat`, and need only
contain the one line:

.. code-block:: shell

    doxypypy -a -c %1

Running Doxygen as usual should now run all Python code through doxypypy.  Be
sure to carefully browse the Doxygen output the first time to make sure that
Doxygen properly found and executed doxypypy.

.. _Doxygen: http://www.stack.nl/~dimitri/doxygen/
.. _doxypy: https://github.com/Feneric/doxypy
.. _PEP 257: http://www.python.org/dev/peps/pep-0257/
.. _Google Python Style Guide: http://google-styleguide.googlecode.com/svn/trunk/pyguide.html?showone=Comments#Comments

