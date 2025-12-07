# QML应用窗口菜单栏设置

```
import QtQuick
import QtQuick.Controls
import QtQuick.Layouts

ApplicationWindow {
    width: 800
    height: 600
    visible: true
    title: qsTr("Test ApplicationWindow")
    id: root
    menuBar: MenuBar {
        id: menubarid
        Menu {
            title: "文件"
            MenuItem {
                text: "打开"
                onClicked: print(text)
            }
            MenuItem {
                text: "退出"
                onClicked: root.close()
            }
        }

        Menu {
            title: "编辑"
            Menu {
                title: "高级"
                MenuItem {
                    text: "剪切行"
                    onClicked: print(text)
                }
            }
        }

        Menu {
            title: "视图"
            MenuItem {
                text: "全屏"
                onClicked: root.showFullScreen()
            }
            MenuItem {
                text: "最大化"
                onClicked: root.showMaximized()
            }

            MenuItem {
                text: "updateMenu"
                onClicked: updateMenu()
            }
        }
    }

    //一级菜单Menu的组件（类型）
    Component {
        id: menucom
        Menu {}
    }

    //动态菜单的事件处理
    property var actions: {
        "updateMenu"//更新菜单
        : updateMenu,
        "close": function () {
            root.close()
        },
        "测试": function () {
            root.close()
        }
    }

    //二级菜单MenuItem的组件（类型）
    Component {
        id: menuitemcom
        MenuItem {
            onTriggered: {
                if (root.actions[text]) {
                    root.actions[text]()
                } else {
                    print("no function " + text)
                }
            }
        }
    }

    function updateMenu() {
        print("updateMenu")

        //菜单的json数据
        var jsonstr = '[{"name":"File","menus":["copy","close"]},{"name":"Menu","menus":["updateMenu","测试"]}]'
        //print(jsonstr)
        //json字符串转换成对象数组
        var menudata = JSON.parse(jsonstr)
        print(menudata[0].name)
        //删除之前的菜单
        while (menubarid.menus.length > 0)
            menubarid.removeMenu(menubarid.menus[0])

        //遍历menudata插入一级菜单
        for (var i = 0; i < menudata.length; i++) {
            //通过组件创建Menu对象
            var menu = menucom.createObject(root)
            menu.title = menudata[i].name

            //插入一级菜单
            menubarid.addMenu(menu)

            //遍历menudata[i].menus 插入二级菜单
            for (var j = 0; j < menudata[i].menus.length; j++) {
                //通过组件创建二级菜单MenuItem
                var item = menuitemcom.createObject(root)
                item.text = menudata[i].menus[j]
                menu.addItem(item)
            }
        }
    }

    //右键菜单
    Menu {
        id: rightMenu
        Menu {
            title: "文件"
            MenuItem {
                text: "打开"
                onClicked: print(text)
            }
            MenuItem {
                text: "退出"
                onClicked: root.close()
            }
        }

        Menu {
            title: "编辑"
            Menu {
                title: "高级"
                MenuItem {
                    text: "剪切行"
                    onClicked: print(text)
                }
            }
        }

        Menu {
            title: "视图"
            MenuItem {
                text: "全屏"
                onClicked: root.showFullScreen()
            }
            MenuItem {
                text: "最大化"
                onClicked: root.showMaximized()
            }

            MenuItem {
                text: "updateMenu"
                onClicked: updateMenu()
            }
        }
    } //Menu

    MouseArea {
        anchors.fill: parent
        acceptedButtons: Qt.RightButton
        onClicked: mouse => {
                       rightMenu.x = mouse.x
                       rightMenu.y = mouse.y
                       rightMenu.open()
                   }
    }

    //工具栏设置
    header: ToolBar {

        RowLayout {
            ToolButton {
                text: "Next"
                onClicked: print("Next")
            }

            ToolButton {
                text: "Last"
                onClicked: print("Last")
            }
            ToolSeparator {}

            ToolButton {
                //text:"Qt"
                icon.source: "qt128.jpg"
                icon.width: 25
                icon.height: 25
                onClicked: print("qt128")
            }
        }
    }

    //TabBar
    footer: TabBar {
        id: tabbar
        TabButton {
            text: "red"
        }

        TabButton {
            text: "lightblue"
        }

        TabButton {
            text: "pink"
        }
    }

    StackLayout {
        currentIndex: tabbar.currentIndex
        Rectangle {
            width: contentItem.width
            height: contentItem.height
            color: "red"
            Text {
                anchors.centerIn: parent
                text: parent.width + "X" + parent.height + " " + parent.color
            }
        }

        Rectangle {
            width: contentItem.width
            height: contentItem.height
            color: "lightblue"
            Text {
                anchors.centerIn: parent
                text: parent.width + "X" + parent.height + " " + parent.color
            }
        }

        Rectangle {
            width: contentItem.width
            height: contentItem.height
            color: "pink"
            Text {
                anchors.centerIn: parent
                text: parent.width + "X" + parent.height + " " + parent.color
            }
        }
    }
} //ApplicationWindow
```