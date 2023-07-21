# Odoo Tutorial

## Module structure

Each module is a directory within a module directory. Module directories are specified by using the `addons-path` option.

An Odoo module is declared by its manifest.

Here is a simplified module directory:

```
module
├── models
│   ├── *.py
│   └── __init__.py
├── data
│   └── *.xml
├── __init__.py
└── __manifest__.py
```

## Enable the developer mode

The developer or debug mode is useful for training as it gives access to additional (advanced) tools. In the next chapters, **we will always assume that you have enabled the developer mode**.



# Odoo的菜单介绍

`<menuitem>` 用于在Odoo的用户界面创建一个新的菜单项

每个`<menuitem>`标签可以有以下几个属性：

- `id`：菜单项的唯一标识符。在整个Odoo系统中，这个ID必须是唯一的。通常，我们会使用模块名作为ID的前缀，以确保其唯一性。
- `name`：菜单项的名称，这是用户在用户界面中看到的名称。
- `parent`：父菜单的ID。如果指定了这个属性，这个菜单项将会作为一个子菜单出现在父菜单下。
- `sequence`：菜单项在同级菜单中的显示顺序。数字越小，菜单项显示的位置越靠前。
- `action`：菜单项点击后执行的操作。这通常是一个到`ir.actions.act_window`对象的引用，该对象定义了当用户点击菜单项时应该打开哪个视图。

```
<odoo>
    <menuitem id="menu_root" name="Library" sequence="10"/>
    <menuitem id="menu_book" name="Books" parent="menu_root" sequence="10" action="action_books"/>
</odoo>
```

在这个例子中，我们创建了两个菜单项：一个顶级菜单`Library`和一个子菜单`Books`。当用户点击`Books`菜单时，将会执行ID为`action_books`的操作。

```
<odoo>
    <record id="action_books" model="ir.actions.act_window">
        <field name="name">Books</field>
        <field name="res_model">library.book</field>
        <field name="view_mode">tree,form</field>
    </record>
</odoo>
```

# Odoo的视图

Odoo的基础视图（basic view）用于定义和展示模型数据的用户界面。在Odoo中，有几种类型的基础视图，每种视图都有其特定的用途和布局。以下是一些最常见的基础视图：

1. **表单视图（Form Views）**：表单视图用于展示和编辑一个模型的单个记录。它通常包含多个字段，这些字段按照一定的布局组织在一起。你可以在表单视图中使用各种控件，如文本框、选择框、日期选择器等。
2. **列表视图（List Views）**：列表视图用于展示一个模型的多个记录。它通常显示为一个表格，每一行对应一个记录，每一列对应一个字段。
3. **看板视图（Kanban Views）**：看板视图用于以卡片的形式展示一个模型的多个记录。每一张卡片对应一个记录，卡片上可以显示记录的一些关键信息。看板视图常常用于任务管理或项目管理等场景。
4. **日历视图（Calendar Views）**：日历视图用于按照日期或时间展示一个模型的多个记录。每个记录显示为日历上的一个事件。
5. **图形视图（Graph Views）**：图形视图用于以图形的形式展示一个模型的多个记录的统计信息。你可以选择使用柱状图、折线图或饼图等不同的图形类型。
6. **枢轴视图（Pivot Views）**：枢轴视图用于展示一个模型的多个记录的交叉报表。你可以选择两个字段作为行和列的维度，然后选择一个字段作为值的度量。

在Odoo中，视图是用来展示模型数据的用户界面。Odoo支持多种类型的视图，包括表单视图（Form）、列表视图（List）、看板视图（Kanban）、图形视图（Graph）、日历视图（Calendar）等。

视图通常在XML文件中定义，并且通过继承和修改的方式可以轻易地定制视图的外观和行为。视图的定义通常包含两部分：视图记录和视图体系结构。

视图记录是使用`<record>`标签定义的，它在Odoo的数据库中创建一个新的`ir.ui.view`记录。`<record>`标签需要以下几个字段：

- `id`：视图的唯一标识符。这个ID在整个Odoo系统中必须是唯一的。
- `model`：视图所属的模型的名称。
- `name`：视图的名称。这个字段是可选的，但是通常会填写，以便于识别和引用视图。

视图体系结构是使用`<field name="arch" type="xml">`标签定义的，它描述了视图的具体结构和样式。视图体系结构的具体内容取决于视图的类型。例如，表单视图通常包含`<form>`、`<sheet>`、`<group>`和`<field>`等标签，而列表视图通常包含`<tree>`和`<field>`等标签。

```
<odoo>
    <record id="view_book_form" model="ir.ui.view">
        <field name="name">book.form</field>
        <field name="model">library.book</field>
        <field name="arch" type="xml">
            <form string="Book">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="author_ids"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="view_book_tree" model="ir.ui.view">
        <field name="name">book.tree</field>
        <field name="model">library.book</field>
        <field name="arch" type="xml">
            <tree string="Books">
                <field name="name"/>
                <field name="author_ids"/>
            </tree>
        </field>
    </record>
</odoo>
```

## 表单视图

表单视图（Form View）主要用于展示和编辑一个模型的单个记录。表单视图通常包含多个字段，这些字段按照一定的布局组织在一起。

表单视图通常在XML文件中定义，使用`<form>`标签。在`<form>`标签内部，可以使用多种标签来定义表单的布局和内容：

- `<sheet>`：定义表单的主体部分。表单中的大部分内容通常放在`<sheet>`标签内部。
- `<group>`：定义一个字段组。字段组可以包含多个字段，并且可以有自己的标签和布局。
- `<field>`：定义一个字段。`<field>`标签的`name`属性指定了字段的名称。你还可以使用其他属性来定义字段的行为和外观，如`readonly`、`required`等。
- `<notebook>`和`<page>`：定义一个标签页组和一个标签页。标签页组可以包含多个标签页，每个标签页可以包含多个字段或字段组。

```
<odoo>
    <record id="view_book_form" model="ir.ui.view">
        <field name="name">book.form</field>
        <field name="model">library.book</field>
        <field name="arch" type="xml">
            <form string="Book">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="author_ids"/>
                    </group>
                    <notebook>
                        <page string="Details">
                            <group>
                                <field name="publish_date"/>
                                <field name="pages"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
</odoo>
```

## 列表视图

列表视图（也被称为树视图）是Odoo中最常见的视图类型之一，它以表格的形式显示记录的列表。每一行代表一个记录，每一列代表一个字段。用户可以点击表头来排序记录，也可以点击每一行来打开表单视图查看或编辑记录的详细信息。

```
<odoo>
    <record id="view_book_tree" model="ir.ui.view">
        <field name="name">book.tree</field>
        <field name="model">library.book</field>
        <field name="arch" type="xml">
            <tree string="Books">
                <field name="name"/>
                <field name="author_id"/>
                <field name="publish_date"/>
            </tree>
        </field>
    </record>
</odoo>
```

## 字段

Odoo提供了多种字段类型，以下是一些最常见的字段类型：

1. **Char**：用于存储短文本字符串，如名称、标题等。
2. **Text**：用于存储长文本字符串，如描述、注释等。
3. **Integer**：用于存储整数。
4. **Float**：用于存储浮点数。
5. **Boolean**：用于存储布尔值，即真或假。
6. **Date** 和 **Datetime**：用于存储日期和日期时间。
7. **Binary**：用于存储二进制数据，如图片、文件等。
8. **Selection**：用于存储预定义的选项列表中的一个选项。
9. **Many2one**、**One2many** 和 **Many2many**：用于定义和其他模型的关联关系。

Selection字段：

在Odoo中，`Selection`字段类型用于创建一个预定义的下拉列表，用户可以从列表中选择一个选项。`Selection`字段的选项列表在代码中定义，并且可以在运行时改变。

`Selection`字段在定义时需要提供一个选项列表。每个选项是一个元组，包含两个元素：第一个元素是选项的技术名称（存储在数据库中），第二个元素是选项的标签（显示给用户）。

```
state = fields.Selection([
    ('draft', 'Draft'),
    ('confirmed', 'Confirmed'),
    ('done', 'Done'),
], 'Status', default='draft')
```


在Odoo中，`Date`和`Datetime`字段类型用于存储日期和日期时间。

1. **Date字段**：用于存储日期，格式为 "YYYY-MM-DD"。
2. **Datetime字段**：用于存储日期和时间，格式为 "YYYY-MM-DD HH:MM:SS"。

