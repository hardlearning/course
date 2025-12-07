# GridView

```
import QtQuick
import QtQuick.Window
import QtQuick.Controls

Window {
    width: 800
    height: 600
    visible: true
    title: qsTr("Test GridView")

    ListModel {
        id: datas
        ListElement {
            name: "testname001"
            number: "1111111111"
        }
        ListElement {
            name: "testname001"
            number: "1111111111"
        }
        ListElement {
            name: "testname002"
            number: "22222222"
        }
        ListElement {
            name: "testname003"
            number: "333333333"
        }
        ListElement {
            name: "testname004"
            number: "444444444"
        }
        ListElement {
            name: "testname005"
            number: "555555555"
        }
        ListElement {
            name: "testname006"
            number: "666666666"
        }
        ListElement {
            name: "testname007"
            number: "777777777"
        }
    }

    Column {
        Row {
            spacing: 10
            Button {
                text: "Icon"
                onClicked: {
                    gridView.delegate = iconDelegate
                    gridView.cellWidth = 100
                    gridView.cellHeight = 100
                }
            }
            Button {
                text: "List"
                onClicked: {
                    gridView.delegate = listDelegate
                    gridView.cellWidth = 200
                    gridView.cellHeight = 50
                }
            }
            Button {
                text: "Edit"
                onClicked: {
                    gridView.delegate = editDelegate
                    gridView.cellWidth = 200
                    gridView.cellHeight = 50
                }
            }
            Button {
                text: "Save"
                onClicked: {
                    output.text = ""

                    for (var i = 0; i < datas.count; i++) {
                        output.text += datas.get(i).name + "\t" + datas.get(i).number
                        output.text += "\n"
                    }
                }
            }
        }
        Text {
            id: output
            text: "output"
        }

        Component {
            id: iconDelegate
            Rectangle {
                implicitWidth: 100
                implicitHeight: 100
                border.color: "#CCCCCC"
                color: gmouse.pressed ? "#777777" : (gmouse.containsMouse ? "#999999" : "#EEEEEE")
                Column {
                    anchors.centerIn: parent
                    Image {
                        source: "qt128.jpg"
                        anchors.horizontalCenter: parent.horizontalCenter
                        width: 40
                        height: 40
                    }
                    Text {
                        text: name
                    }
                    Text {
                        text: number
                    }
                }
                MouseArea {
                    id: gmouse
                    anchors.fill: parent
                    hoverEnabled: true
                    onClicked: {
                        print(datas.get(index).name)
                        print(datas.get(index).number)
                        print("index = " + index)
                    }
                }
            }
        }

        Component {
            id: listDelegate
            Rectangle {
                implicitWidth: gridView.cellWidth
                implicitHeight: gridView.cellHeight
                Row {
                    spacing: 10
                    anchors.centerIn: parent
                    Text {
                        text: index + 1
                    }
                    Text {
                        text: name
                    }
                    Text {
                        text: number
                    }
                }
            }
        }

        Component {
            id: editDelegate
            Rectangle {
                implicitWidth: gridView.cellWidth
                implicitHeight: gridView.cellHeight
                Row {
                    spacing: 10
                    anchors.centerIn: parent
                    TextField {
                        text: name
                        onEditingFinished: name = text
                    }
                    TextField {
                        text: number
                        onEditingFinished: number = text
                    }
                }
            }
        }

        GridView {
            id: gridView
            width: 300
            height: 200
            cellWidth: 200
            cellHeight: 50
            model: datas
            delegate: listDelegate
        } //GridView
    } //Column
}
```