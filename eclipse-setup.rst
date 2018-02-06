****************************
Eclipse IDE 的创建和烧录指南
****************************

.. _eclipse-install-steps:

安装 Eclipse IDE
================

Eclipse IDE 是一个可视化的集成开发环境，可用于编写、编译和调试 ESP-IDF 项目。

* 首先，请在您的平台上安装相应的 ESP-IDF，具体步骤请参考适用于 Windows、OS X 和 Linux 的相应安装步骤。

* 我们建议，您应首先使用命令行创建一个项目，大致熟悉项目的创建流程。此外，您还需要使用命令行 (``make menuconfig``) 对您的 ESP-IDF 项目进行配置。目前，Eclipse 尚不支持 ESP-IDF 项目的配置。

* 下载 Eclipse Installer 至您的平台，点击 :ref: eclipse.org_。

* 运行 Eclipse Installer 时，选择 `Eclipse for C/C++ Development`（有的版本也可能显示为 CDT）。

Windows 用户
============

Eclipse IDE 的 Windows 用户，请参考 :ref: Windows 用户的 Eclipse IDE 使用指南<eclipse-windows-setup>

配置 Eclipse IDE
=================

请打开安装好的 Eclipse IDE，并按照以下步骤进行操作：

导入新项目
----------

* Eclipse IDE 需使用 ESP-IDF 的 ``Makefile`` 功能。因此，在使用 Eclipse 前，您需要先创建一个 ESP-IDF 项目。在创建 ESP-IDF 项目时，您可以使用 GitHub 中的 idf-template 项目模版，或从 esp-idf 子目录中选择一个 example。

* 运行 Eclipse，选择 `File`->`Import...`。

* 在弹出的对话框中选择 `C/C ++`->`Existing Code as Makefile Project`，然后点击 `Next`。

* 跳转至下个界面，在 `Existing Code Location` 位置输入您的 IDF 项目的路径。注意，这里应填入 ESP-IDF 项目的路径，而非 ESP-IDF 的路径（稍后再填写）。此外，您指定的目标路径中应包含名为 `Makefile`（项目 Makefile）的文件。

* 在本界面，选择 `Toolchain for Indexer Settings`，并选择 `Cross GCC`，最后点击 `Finish`。


项目属性
--------

* 新项目将出现在 `Project Explorer` 下。请右键选择该项目，并在菜单中选择 `Properties`。

* 点击 `C/C++ Build` 下的 `Environment` 属性页，选择 `Add...`，并在对应位置输入 `BATCH_BUILD` 和 `Value 1`。

* 再次点击 `Add...`，并在 `IDF_PATH` 中输入 `ESP-IDF` 所在的完整安装路径。

* 选择 `PATH` 环境变量，不要改变默认值。如果 IDF 设置中的 Xtensa 工具链尚不在 `PATH` 列表中，则应将该路径增加至列表 (`something/xtensa-esp32-elf/bin`)。

* 在 macOS 平台中，增加 `PYTHONPATH` 环境变量，并将其设置为 `/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages`，允许预先安装的`系统 Python` 可以覆盖任何内置的 `Eclipse Python`。

* 前往 `C/C ++General`->`Preprocessor Include Paths` 属性页面。

* 点击 `Providers` 选项卡。从 `Providers` 列表中选择 `CDT Cross GCC Built-in Compiler Settings`。进入 `Command to get compiler specs` 目录，用 `xtensa-esp32-elf-gcc` 替换行首的 `text${COMMAND}`，最终的完整 `Command to get compiler specs` 应为 `xtensa-esp32-elf-gcc${FLAGS}-E-P-v-dD“${INPUTS}`。

* 从 `Providers` 列表中选择 `CDT GCC Build Output Parser`，然后在 Compiler 命令模式的起始位置输入 `xtensa-esp32-elf-`，最终的完整编译器命令应为 `xtensa-esp32-elf-(g？cc)|([gc]\+\+)|(clang)`

.. _eclipse-build-project:

在 Eclipse IDE 中创建项目
--------------------------

在首次创建项目前，Eclipse IDE 可能会显示大量有关未定义值的错误和警告，主要原因在于项目编译过程中所需的一些源文件是在 ESP-IDF 项目新建过程中自动生成的。因此，这些错误和警告将在 ESP-IDF 项目生成完成后消失。

* 点击 `OK`，关闭 Eclipse IDE 中的 `Properties` 对话框。

* 在 Eclipse IDE 界面外，打开命令管理器。进入项目目录，并通过 `make menuconfig` 命令对您的 ESP-IDF 项目进行配置。现阶段，Eclipse 还不能进行这一步操作。

*如果您跳过最开始的配置步骤，ESP-IDF 将在命令行中提示进行配置 - 但由于 Eclipse 暂时不支持相关功能，因此该项目将挂起或失败。*

* 返回 Eclipse IDE 界面中，选择 `Project`->`Build` 建立您的项目。

**提示**：如果您已经在 Eclipse IDE 环境外创建了项目，则可能需要选择 `Project`->`Clean before choosing Project`->`Build`，允许 Eclipse 查看所有源文件的编译器参数，并借此确定头文件包含路径。

在 Eclipse IDE 中烧录项目
--------------------------

您可以将 `make flash` 目标放在 Eclipse 项目中，使用 Eclipse UI 中的 `esptool.py` 进行烧录：

* 打开 `Project Explorer`，并右击您的项目（请注意右击项目本身，而非项目下的子文件，否则 Eclipse 可能无法找到正确的 `Makefile`）。

* 从菜单中选择 `Make Targets`->`Create`。

* 输入 `flash` 为目标名称，其他选项使用默认值。

* 选择 `Project`->`Make Target`->`Build (快捷键：Shift + F9）` ，创建自定义烧录目标，用于编辑、烧录项目。

注意，您将需要通过 `make menuconfig`，设置串行端口和其他烧录选项。`make menuconfig` 仍需通过命令操作（请见平台的对应指南）。

如有必要，请按照相同步骤添加 `bootloader` 和 `partition_table`。

相关文档
--------

.. toctree::
    :maxdepth: 1

    eclipse-setup-windows


.. _eclipse.org: https://www.eclipse.org/


