# 创建窗口的三种方法

- NewWindow.qml

```
import QtQuick

Window {
    width: 300
    height: 300
    title: "New Window"
}
```

- Main.qml

```
import QtQuick
import QtQuick.Controls

//主窗口
Window {
    width: 800
    height: 600
    visible: true
    title: qsTr("Test Window")
    id: root

    //子窗口
    Window {
        id: subwin
        width: 400
        height: 300
        title: "Sub Window"
    }
    NewWindow {
        id: newwin
    }

    //通过组件创建窗口
    Component {
        id: wincom
        Window {
            width: 400
            height: 300
            title: "wincom subwin"
        }
    }

    Column {
        Row {
            spacing: 10
            Text {
                text: "创建窗口"
            }

            Button {
                text: "子窗口"
                onClicked: subwin.show()
            }

            Button {
                text: "NewWindow"
                onClicked: newwin.show()
            }

            Button {
                text: "Component subwin"
                onClicked: {
                    var win = wincom.createObject()
                    win.show()
                }
            }

            Button {
                text: "createComponent subwin"
                onClicked: {
                    //创建Component
                    var com = Qt.createComponent("NewWindow.qml")
                    var win = com.createObject(root)
                    win.title = text
                    win.show()
                }
            }
        }
        Row {
            spacing: 10
            Text {
                text: "设置窗口"
            }
            Button {
                text: "最大化"
                onClicked: root.showMaximized()
            }
            Button {
                text: "全屏"
                onClicked: root.showFullScreen()
            }
            Button {
                text: "Dialog"
                // 查询 Qt::WindowFlags Qt::Dialog=>Qt.Dialog
                onClicked: root.flags = Qt.Dialog
            }

            Button {
                text: "Window"
                onClicked: root.flags = Qt.Window
            }

            Button {
                text: "FramelessWindowHint"
                onClicked: root.flags = Qt.FramelessWindowHint
            }
        }
        Row {
            spacing: 10
            Text {
                text: "设置窗口"
            }

            //模态窗口


            /*
                Qt.NonModal(默认)
                Qt.WindowModal
                Qt.ApplicationModal
            */
            Button {
                text: "模态窗口 WindowModal"
                onClicked: {
                    subwin.modality = Qt.WindowModal
                    subwin.show()
                }
            }

            Button {
                text: "模态窗口 NonModal"
                onClicked: {
                    subwin.modality = Qt.NonModal
                    subwin.show()
                }
            }

            Button {
                text: "模态窗口 ApplicationModal"
                onClicked: {
                    subwin.modality = Qt.ApplicationModal
                    subwin.show()
                }
            }
        }
    }
}
```