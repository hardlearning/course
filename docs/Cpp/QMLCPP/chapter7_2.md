# QML文件打开和保存Dialog

```
import QtQuick
import QtQuick.Controls.Fusion
//需要放在import QtQuick.Dialogs之前
import QtQuick.Dialogs
import QtQuick.Controls

//import Qt.labs.platform
Window {
    width: 800
    height: 600
    visible: true
    title: qsTr("Test Dialog")

    //MessageDialog
    //QML Rectangle: The current style does not support
    MessageDialog {
        id: md1
        text: "MessageDialog text"
        title: "MessageDialog title"
        buttons: MessageDialog.Ok | MessageDialog.Close
        onAccepted: print("MessageDialog onAccepted")
        onRejected: print("MessageDialog onRejected")
    }

    FileDialog {
        id: openfile
        //当前文件
        //currentFile
        //当前目录
        //folder

        //FileDialog.OpenFile
        //FileDialog.OpenFiles
        //FileDialog.SaveFile
        fileMode: FileDialog.OpenFile
        acceptLabel: "上传"
        onAccepted: {
            md1.text = selectedFile
            md1.open()
        }
    }

    FileDialog {
        id: savefile
        fileMode: FileDialog.SaveFile
        onAccepted: {
            md1.text = selectedFile
            md1.open()
        }
    }

    Row {
        spacing: 10
        padding: 10
        Button {
            text: "MessageDialog"
            onClicked: md1.open()
        }

        Button {
            text: "Open File"
            onClicked: openfile.open()
        }
        Button {
            text: "Save File"
            onClicked: savefile.open()
        }
    }
}
```