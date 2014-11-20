fetch-image-from-database
=========================
Database mysql

import glob
import multiprocessing
import time 
import MySQLdb
import ast
from datetime import datetime
import os
import sys
import req_proxy
import logging
import time
import multiprocessing
import os
import code1_first_image_call

logging.basicConfig(level=logging.DEBUG, format='[%(levelname)s] (%(threadName)-10s) %(message)s')

num_fetch_threads = 10
enclosure_queue = multiprocessing.JoinableQueue()



def my_strip(x):
    try:
        x = str(x).strip()
        x = MySQLdb.escape_string(x).strip()
    except:
        x = str(x.encode("ascii", "ignore")).strip()
        x = MySQLdb.escape_string(x).strip()

    return x



def mainthread2(i, q):
    for directory,  link in iter(q.get, None):
        try:
           #link = link[0]
           #logging.debug(link)
           code1_first_image_call.main(directory, link)

 

        except:
            fname  ="%s/code_first_fetch_imgage_error.txt" %(directory)
            f2 = open(fname, "a+")
            f2.write(str(link).strip() +"\n")
            f2.close()

        time.sleep(2)
        q.task_done()

    q.task_done()




def supermain():
    directory = "/home/desktop/working_sites/mirraw_site/mirraw/image_folder"

    try:
        os.makedirs(directory)
    except:
        pass

    db = MySQLdb.connect("192.99.13.229","root","6Tresxcvbhy","mirrow")
    cursor = db.cursor()


    sql_select = """select task_id, image_link from mirrow_data where upload_image_status = 'NO' """
    cursor.execute(sql_select)

    results = cursor.fetchall()

    procs = []

    for i in range(num_fetch_threads):
        procs.append(multiprocessing.Process(target=mainthread2, args=(i, enclosure_queue,)))
        procs[-1].start()

    for link in results:
        enclosure_queue.put((directory, link))

    enclosure_queue.join()

    for p in procs:
        enclosure_queue.put(None)

    enclosure_queue.join()

    for p in procs:
        p.join(120)

    sql = """UPDATE  mirrow_data  set upload_image_status = 'YES' """
    cursor.execute(sql)
    db.commit()

    db.close()


if __name__=="__main__":
        supermain()
   
