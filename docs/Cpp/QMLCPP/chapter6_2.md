# TableView

```
import QtQuick
import QtQuick.Window
import Qt.labs.qmlmodels
import QtQuick.Controls

Window {
    width: 800
    height: 600
    visible: true
    title: qsTr("Test TableView")
    
    TableModel {
        id: tableModel
        TableModelColumn {
            display: "check"
        }
        TableModelColumn {
            display: "name"
        }
        TableModelColumn {
            display: "color"
        }
        rows: [{
                "check": true,
                "name": "cat",
                "color": "black"
            }, {
                "check": false,
                "name": "dog",
                "color": "brown"
            }, {
                "check": true,
                "name": "bird",
                "color": "white"
            }, {
                "check": true,
                "name": "sssss",
                "color": "pink"
            }, {
                "check": true,
                "name": "aaaa",
                "color": "yellow"
            }]
    }
    
    Item {
        anchors.fill: parent
        //width:parent.width
        //height: parent.height
        //columns: 2
        Rectangle {
            border.width: 1
            color: "#EEEEEE"
            anchors.left: parent.left
            anchors.right: parent.horizontalCenter
            anchors.top: parent.top
            anchors.bottom: parent.verticalCenter
            
            /// 1基础示例
            TableView {
                model: tableModel
                anchors.fill: parent
                delegate: Text {
                    text: display
                }
            }
        } //Rectangle
        
        Rectangle {
            border.width: 1
            color: "#EEEEEE"
            anchors.left: parent.horizontalCenter
            anchors.right: parent.right
            anchors.top: parent.top
            anchors.bottom: parent.verticalCenter
            
            /// 2编辑代理和界面设置选中及事件处理
            TableView {
                id: tableView2
                model: tableModel
                anchors.fill: parent
                columnSpacing: 1
                rowSpacing: 1
                selectionModel: ItemSelectionModel {
                    onCurrentChanged: (current, previous) => {
                        print("onCurrentChanged = " + current + ":" + previous)
                    }
                }
                
                //row column display 代理可以直接访问
                delegate: Rectangle {
                    implicitWidth: 100
                    implicitHeight: 40
                    color: tableView2.currentRow == row ? "#999999" : "#DDDDDD"
                    Text {
                        anchors.centerIn: parent
                        text: display
                    }
                    TableView.editDelegate: TextField {
                        anchors.fill: parent
                        text: display
                        horizontalAlignment: TextField.AlignHCenter
                        verticalAlignment: TextField.AlignVCenter
                        //默认选中文本
                        Component.onCompleted: selectAll()
                        TableView.onCommit: display = text
                    }
                }
            }
        } //Rectangle
        
        Rectangle {
            border.width: 1
            color: "#EEEEEE"
            anchors.left: parent.left
            anchors.right: parent.horizontalCenter
            anchors.top: parent.verticalCenter
            anchors.bottom: parent.bottom
            
            //3 单列设置编辑方式
            TableView {
                model: tableModel
                anchors.fill: parent
                delegate: DelegateChooser {
                    DelegateChoice {
                        column: 0
                        delegate: Rectangle {
                            implicitWidth: 100
                            implicitHeight: 30
                            CheckBox {
                                anchors.centerIn: parent
                                checked: model.display
                                onToggled: model.display = checked
                            }
                        }
                    }
                    
                    DelegateChoice {
                        column: 2
                        delegate: Rectangle {
                            implicitWidth: 100
                            implicitHeight: 30
                            TextInput {
                                anchors.centerIn: parent
                                text: model.display
                                onAccepted: model.display = text
                                onEditingFinished: model.display = text
                            }
                        }
                    }
                    
                    //默认显示方式
                    DelegateChoice {
                        delegate: Rectangle {
                            implicitWidth: 100
                            implicitHeight: 30
                            Text {
                                text: model.display
                            }
                        }
                    }
                }
            }
        } //Rectangle
        
        Rectangle {
            border.width: 1
            color: "#EEEEEE"
            
            anchors.left: parent.horizontalCenter
            anchors.right: parent.right
            anchors.top: parent.verticalCenter
            anchors.bottom: parent.bottom
            
            //4 设置标题栏，点击标题栏排序数据
            //水平标题
            HorizontalHeaderView {
                id: hhead
                anchors.left: tableView4.left
                syncView: tableView4
                property bool desc: true
                //model:["check","name","color"]
                delegate: Rectangle {
                    implicitWidth: 100
                    implicitHeight: 30
                    color: "lightblue"
                    Text {
                        anchors.centerIn: parent
                        text: tableModel.columns[index].display + " " + (hhead.desc ? "↑" : "↓")
                    }
                    MouseArea {
                        anchors.fill: parent
                        onClicked: {
                            
                            var key = tableModel.columns[index].display
                            
                            //冒泡排序数据
                            for (var i = tableModel.rowCount - 1; i > 0; i--) {
                                for (var j = 0; j < i; j++) {
                                    if (hhead.desc) {
                                        if (tableModel.getRow(
                                                    j)[key] > tableModel.getRow(
                                                    j + 1)[key]) {
                                            tableModel.moveRow(j, j + 1, 1)
                                        }
                                    } else {
                                        if (tableModel.getRow(
                                                    j)[key] < tableModel.getRow(
                                                    j + 1)[key]) {
                                            tableModel.moveRow(j, j + 1, 1)
                                        }
                                    }
                                }
                            }
                            
                            hhead.desc = !hhead.desc
                        }
                    }
                }
            }
            
            //垂直标题
            VerticalHeaderView {
                id: vhead
                anchors.top: tableView4.top
                syncView: tableView4
                delegate: Rectangle {
                    implicitWidth: 20
                    implicitHeight: 20
                    color: "lightblue"
                    Text {
                        anchors.centerIn: parent
                        text: display
                    }
                }
            }
            
            
            /*
              HHHH
            V TTT
            V TTT
            V TTT
            */
            TableView {
                id: tableView4
                model: tableModel
                anchors.left: vhead.right
                anchors.right: parent.right
                anchors.top: hhead.bottom
                anchors.bottom: parent.bottom
                columnSpacing: 1
                rowSpacing: 1
                //anchors.fill: parent
                delegate: Rectangle {
                    implicitWidth: 100
                    implicitHeight: 30
                    Text {
                        anchors.centerIn: parent
                        text: model.display
                    }
                }
            }
        } //Rectangle
    } //Gird
}
```