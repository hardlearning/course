# ListView

```
import QtQuick
import QtQuick.Controls

Window {
    width: 800
    height: 600
    visible: true
    title: qsTr("Test ListView")

    Column {
        padding: 5
        ////新增 修改  删除
        Text {
            id: errtxt
            color: "red"
        }
        Row {
            Text {
                text: "name:\t"
            }
            TextField {
                id: nameinput
            }
        }
        Row {
            Text {
                text: "number:\t"
            }
            TextField {
                id: numberinput
            }
        }
        Row {
            padding: 5
            spacing: 10
            Button {
                text: "新增"
                onClicked: {
                    if (nameinput.text.length <= 0 || numberinput.text.length <= 0) {
                        errtxt.text = "请输入name和number！"
                        return
                    }
                    errtxt.text = ""
                    datas.append({
                        "name": nameinput.text,
                        "number": numberinput.text
                    })
                }
            }
            Button {
                text: "修改"
                onClicked: {
                    if (nameinput.text.length <= 0 || numberinput.text.length <= 0) {
                        errtxt.text = "请输入name和number！"
                        return
                    }
                    errtxt.text = ""

                    datas.set(listView.currentIndex, {
                        "name": nameinput.text,
                        "number": numberinput.text
                    })
                }
            }
            Button {
                text: "删除"
                onClicked: {
                    if (listView.currentIndex < 0 || datas.count <= 0)
                        return
                    datas.remove(listView.currentIndex)
                }
            }
        }

        //滚动条
        ScrollView {
            width: 300
            height: 400
            //View
            ListView {
                id: listView
                anchors.fill: parent

                //头部设置
                header: Rectangle {
                    width: parent.width
                    height: 40
                    color: "lightblue"
                    Text {
                        anchors.centerIn: parent
                        text: "====== header ====== "
                        font.pixelSize: 18
                    }
                }

                //底部
                footer: Rectangle {
                    width: parent.width
                    height: 40
                    color: "lightblue"
                    Text {
                        anchors.centerIn: parent
                        text: "====== footer ====== "
                        font.pixelSize: 18
                    }
                }

                //选中的样式
                highlight: Rectangle {
                    width: parent.width
                    color: "lightgray"
                }

                //数据模型
                model: ListModel {
                    id: datas
                    ListElement {
                        name: "用户名001"
                        number: "10000001"
                    }
                    ListElement {
                        name: "用户名002"
                        number: "10000002"
                    }
                    ListElement {
                        name: "用户名003"
                        number: "10000003"
                    }
                }

                //代理
                //遍历ListModel 直接通过key访问ListElement 数据
                //index读取ListModel数组下标
                delegate: Item {
                    width: listView.width
                    height: 30
                    Row {
                        spacing: 5
                        Image {
                            source: "qt128.jpg"
                            width: 20
                            height: 20
                        }
                        Text {
                            text: index + " " + name + ":" + number
                            font.pixelSize: 16
                        }
                    }
                    MouseArea {
                        anchors.fill: parent
                        onClicked: {
                            //选中数据改变样式
                            listView.currentIndex = index
                            //读取数据用于修改
                            nameinput.text = name
                            numberinput.text = number
                            //print("onClicked")
                        }
                    }
                }
            } //ListView
        } //ScrollView
    } //Column
}
```