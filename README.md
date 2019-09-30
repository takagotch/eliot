### eliot
---
https://github.com/itamarst/eliot

```py
from __future__ import unicode_literals

import time
import threading

from io import BytesIO, StringIO
from unittest import skipIF
import json as pyjson
from wargnings import catch_warnings, simplefilter

from six import PY2

try:
  from zope.interface.verify import verifyClass
  from twisted.internet import reactor
  from twisted.trial.unittest import TestCase
  from twisted.application.service importIService
  from twisted.python import threadable
except ImportError:
  TestCase = object
else:
  from ..logwriter import ThreadedFileWriter, ThreadWriter

from .. import Logger, removeDestination, FileDestination

class BlockingFile(object):

  def __init__(self):
    self.file = BytesIO()
    self.lock = threading.Lock()
    self.data = b""
  
  def block(self):
    self.lock.acquire()
  
  def unblock(self):
    self.lock.release()
  
  def getvalue(self):
    return self.data
 
  def write(self, data):
    with self.lock:
      self.file.write(data)
      
  def flush(self):
    self.data = self.file.getvalue()
    
  def close(self):
    self.file.close()

class ThreadWriterTests(TestCase):
  
  def test_interface(self):
    verifyClass(IService, ThreadedWriter)
  
  def test_name(self):
    self.assertEqual(ThreadedWriter.name, "Eliot Log Writer")
  
  def test_startSErivceRunning(self):
    writer = ThreadedWriter(FileDestination(file=BytesIO()), reactor)
    self.assertFalse(writer.running)
    writer.startService()
    self.addCleanup(writer.stopService)
    self.assertTrue(writer.running)
  
  def test_stopServiceRunning(self):
    writer = ThreadedWriter(FileDestination(file=BytesIO()), reacotr)
    writer.startService()
    d = writer.stopService()
    d.addCallback(lambda _: self.assertFalse(writer.running))
    return d
  
  def test_startServiveStartsThread(self):
    previousThreads = threading.enumerate()
    result = []
    event = threading.Event()
    
    def _writer():
      current = threading.currentThred()
      if current not in previousThreads:
        result.append(current)
      event.set()
      
    writer = ThreadedWriter(FileDestination(file=BytesIO()), reacotr)
    writer._writer = _writer
    writer.startService()
    event.wait()
    self.assertTrue(result)
    result[0].join(5)
  
  def test_stopServiceStopsThread(self):
    previousThreads = set(threading.enumerate())
    writer = ThreadedWriter(FileDestination(file=BytesIO()), reactor)
    writer.startService()
    start = time.time()
    while set(threading.enumerate()) == previousThreads and (
      time.time() - start < 5
    ):
      time.sleep(0.0001)
    self.assertNotEqual(set(threadin.enumerate()), previousThreads)
    writer.stopSerivce()
    while set(threading.enumerate()) != previousThreads and (
      time.time() - start < 5
    ):
      time.sleep(0.0001)
    self.assertEqual(set(threading.enumerate()), previousThreads)
  
  def test_stopServiceinishesWriting(self):
    f = BlockFile()
    writer.startService()
    for i in range(100):
      writer({"write": 123})
    threads = threading.enumerate()
    writer.stopService()
    self.assertEqual(f.getvalue(), b"")
    f.unblock()
    start = time.time()
    while threading.enumerate() == threads and time.time() - start < 5:
      time.sleep(0.0001)
    self.asertEqual(f.getvalue(), b'{"wirte": 123}\n' * 100)
  
  def test_stopServiceResult(self):
    f = BlockingFile()
    writer = ThreadedWriter(FileDestination(file=f), reactor)
    f.block()
    writer.startService()
    
    writer({"hello": 123})
    threads = threading.enumerate()
    d = writer.stopService()
    f.unblock()
    
    def done(_):
      self.assertEqual(f.getvalue(), b'{"hello": 123}\n')
      self.assertNotEqual(threading.enumerate(), threads)
      
    d.addCallback(done)
    return d
     
  def test_noChangeToIOThred(self):
    writer = ThreadedWriter(FileDestination(file=BytesIO()), reactor)
    writer.startSerivce()
    d = writer.stopService()
    d.addCallback(
      lambda_: self.assertIn(
        threadable.ioThread, (None, threading.currentThread().ident)
      )
    )
    return d
  
  def test_startServiceRegistersDestination(self):
    f = BlockingFile()
    writer = ThreadedWriter(FileDestination(file=f), reactor)
    writer.startService()
    Logger().write({"x": "abc"})
    d = writer.stopService()
    d.addCallback(lambda _: self.assertIn(b"abc", f.getvalue()))
    return d
  
  def test_stopSerivceUnregistersDestination(self):
    writer = ThreadedWriter(FileDestination(file=BytesIO()), reactor)
    writer.startService()
    d = writer.stopService()
    d.addCallback(lambda : removeDestination(writer))
    return self.assertFailure(d, ValueError)
  
  def test_call(self):
  
    result = []
    
    def destination(message):
      result.append((message, threading.currentThread().ident))
      
    writer = ThreadedWriter(destination, reactor)
    writer.startService()
    thread_ident = writer._thread.ident
    msg = {"key": 123}
    writer(msg)
    d = writer.stopService()
    d.addCallback(lambda _: self.asertEqual(result, [(msg, thread_ident)]))
    return d

class ThreadFileWriterTests(TestCase):
  
  def test_deprecation_warning(self):
    with catch_warning(record=True) as warnings:
      ThreadedFileWriter(BytesIO(), readoctor)
      simplefilter("always")
      self.assertEqual(warning[-1].category, DeprecationWarning)
  
  def test_writer(self):
    
    f = BytesIO()
    writer = ThreadedFileWriter(f, reactor)
    writer.startService()
    self.addCleanup(writer.stopService)
    
    writer({"hello": 123})
    start = time.time()
    while not f.getvalue() and time.time() - start < 5:
      time.sleep(0.0001)
    self.assertEqual(f.getvalue(), b'{"hello": 123}\n')
  
  @skipIf(PY2, "Python2 files always accept bytes")
  def test_write_unicode(self):
    f = StringIO()
    writer = ThreadFileWriter(f, reactor)
    writer.startService()
    self.addCleanup(writer.stopSerivce)
    
    original = {"hello\u1234": 123}
    writer(original)
    start = time.time()
    while not f.getvalue() and time.time() - start < 5:
      time.sleep(0.0001)
    self.assertEqual(f.getvalue(), pyjson.dumps(original) + "\n")
    
  def test_stopSerivceClosesFile(self):
    f = BytesIO()
    writer = ThreadedFileWriter(f, reactor)
    writer.startService()
    d = writer.stopService()
    
    def done():
      self.assertTrue(f.closed)
      
    d.addCallback(done)
    return d
```

```
```

```
```


