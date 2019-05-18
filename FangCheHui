#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import os
import re
import shutil
import time
import itchat
from itchat.content import *


# 定义一个字典，保存消息的信息。
msg_dict = {}

# 创建一个目录，用于存放消息临时文件。
rec_tmp_dir = "/Users/matianhe/itchat/rec_tmp/"
if not os.path.exists(rec_tmp_dir):
     os.mkdir(rec_tmp_dir)


face_bug = None


# 注册消息接收器
@itchat.msg_register([TEXT, PICTURE, MAP, CARD,             SHARING, RECORDING,
                       ATTACHMENT, VIDEO])
def handler_receive_msg(msg):
     global face_bug
     msg_time_rec = time.strftime("%Y-%m-%d %H:%M%S", time.localtime())
     msg_id = msg['MsgId']
     msg_time = msg['CreateTime']
     msg_from = (itchat.search_friends(userName=msg['FromUserName']
                                       ))["NickName"]
     msg_content = None
     msg_share_url = None

     if msg['Type'] == 'Text' or msg['Type'] == 'Friends':
         msg_content = msg['Text']
     elif msg['Type'] == 'Recording' or msg['Type'] == 'Attachment' \
             or msg['Type'] == 'Video' or msg['Type'] == 'Picture':
         msg_content = r"" + msg['FileName']
         msg['Text'](rec_tmp_dir + msg['FileName'])
    elif msg['Type'] == 'Card':
         msg_content = msg['RecommendInfo']['NickName'] + r" 的名片"
    elif msg['Type'] == 'Map':
         x, y, location = re.search("<location x=\"(.*?)\" y=\"(.*?\".*lable= \
                                    \"(.*?)\".*", msg['OriContent']).group(1, 2,
                                                                           3)
         if location is None:
             msg_content = r"纬度->" + x.__str__() + " 经度->" + y.__str__()
         else:
             msg_content = r"" + location
     elif msg['Type'] == 'Sharing':
         msg_content = msg['Text']
         msg_share_url = msg['Url']
     face_bug = msg_content

     msg_dict.update({
         msg_id: {
             "msg_from": msg_from, "msg_time": msg_time,
             "msg_time_rec": msg_time_rec, "msg_type": msg["Type"],
             "msg_content": msg_content, "msg_share_url": msg_share_url
         }
     })


@itchat.msg_register([NOTE])
def send_msg_helper(msg):
    global face_bug
     if re.search(r"\<\!\[CDATA\[.*撤回了一条消息\]\]\>", msg['Content']) \
             is not None:
         old_msg_id = re.search("\<msgid\>(.*?)\<\/msgid\>", \
                                 msg['Content']).group(1)
         old_msg =msg_dict.get(old_msg_id, {})
         if len(old_msg_id) < 11:
             itchat.send_file(rec_tmp_dir + face_bug,
                                toUserName='filehelper')
            os.remove(rev_tmp_dir + face_bug)
         else:
             msg_body = "有人撤回消息" + "\n" \
                 + old_msg.get('msg_from') + " 撤回了 " \
                 + old_msg.get('msg_type') + " 消息" + "\n" \
                 + old_msg.get('msg_time_rec') + "\n" \
                 + r"" + old_msg.get('msg_content')
             if old_msg['msg_type'] == "Sharing":
                 msg_body += "\n就是这个连接->" + old_msg.get('msg_share_url')
             itchat.send(msg_body, toUserName='filehelper')
             if old_msg["msg_type"] == "Picture" \
                     or old_msg["msg_type"] == "Recording" \
                     or old_msg["msg_type"] == "Video" \
                     or old_msg["msg_type"] == "Attachment":
                 file = '@fil@%s' % (rec_tmp_dir + old_msg['msg_content'])
                 itchat.send(msg=file, toUserName='filehelper')
                 os.remove(rec_tmp_dir + old_msg['msg_content'])
             msg_dict.pop(old_msg_id)


 if __name__ == "__main__":
     itchat.auto_login(hotReload=True, enableCmdQR=2)
     itchat.run()
