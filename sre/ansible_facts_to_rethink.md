# Writing Ansible Facts to RethinkDB

Little did I know when I woke up this morning that I'd be writing a plugin for Ansible. Well, close enough. I have a
microservice that takes a hostname as a JSON body and then uses the
[Ansible API](https://docs.ansible.com/ansible/latest/dev_guide/developing_api.html) to write the facts to a RethinkDB
instance.

To get this to work, I had to modify `ResultCallback()` to write facts to Rethink. Much fun was had. Below is the code as
it stands, sans the many hours of prettying up that need to happen to make it a viable service (error handling, exception
trapping etc):

```python
#!/usr/bin/env python

import codecs
from datetime import datetime
import json
import logging
import os
import sys
import tempfile
import time
from collections import namedtuple

from ansible.parsing.dataloader import DataLoader
from ansible.vars.manager import VariableManager
from ansible.inventory.manager import InventoryManager
from ansible.playbook.play import Play
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.plugins.callback import CallbackBase

import falcon
import rethinkdb as r

logger = logging.getLogger(__name__)

class ResultCallback(CallbackBase):
    """Callback plugin."""

    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'rethinkdb'
    CALLBACK_NEEDS_WHITELIST = False

    REQUEST_HEADERS = {
        "Content-Type": "application/json",
        "Accept": "application/json"
    }
    TIME_FORMAT = "%Y-%m-%d %H:%M:%S %f"

    def send_facts(self, host, data):
        data["_type"] = "ansible"
        data["_timestamp"] = datetime.now().strftime(self.TIME_FORMAT)
        facts = {"name": host, "facts": data}

        try:
            r.connect("localhost", 32769).repl()
            r.db("horde_db").table("inventory").insert([facts]).run()
        except Exception as err:
            print(str(err))

    def v2_runner_on_ok(self, result):
        # host = result._host
        # data = json.dumps({host.name: result._result})
        res = result._result
        module = result._task.action

        if module == 'setup' or 'ansible_facts' in res:
            host = result._host.get_name()
            self.send_facts(host, res)
        else:
            self.append_result(result)


    def v2_runner_on_failed(self, result, **kwargs):
        pass

    def v2_runner_on_unreachable(self, result, **kwargs):
        pass

    def v2_runner_on_skipped(self, result, **kwargs):
        pass

def max_body(limit):

    def hook(req, resp, resource, params):
        length = req.content_length
        if length is not None and length > limit:
            msg = ('The size of the request is too large. The body must not exceed ' + str(limit) + ' bytes in length.')
            raise falcon.HTTPRequestEntityTooLarge('Request body is too large', msg)

    return hook


class HordeCollector(object):

    @falcon.before(max_body(64 * 1024))
    def on_post(self, req, resp):

        data = req.bounded_stream.read()
        data = json.loads(data.decode('utf-8'))
        host = data['hostname']

        fd, path = tempfile.mkstemp()
        with os.fdopen(fd, 'w') as tmp:
            tmp.write(host)

        Options = namedtuple('Options', ['connection', 'module_path', 'forks', 'become', 'become_method', 'become_user', 'check', 'diff'])

        # Initialize needed objects
        loader = DataLoader()
        options = Options(connection='ssh', module_path='', forks=100, become=None, become_method=None, become_user=None, check=False, diff=False)
        passwords = dict()

        results_callback = ResultCallback()

        inventory = InventoryManager(loader=loader, sources=['{0}'.format(path),])
        variable_manager = VariableManager(loader=loader, inventory=inventory)

        # Create a play with no tasks, as we're only interested in the gather_facts process.
        play_source = dict(
            name = "Inventory Tools",
            hosts = 'all',
            gather_facts = 'yes',
            tasks = [])
        play = Play().load(play_source, variable_manager=variable_manager, loader=loader)

        # Run the play
        tqm = None
        try:
            tqm = TaskQueueManager(
                inventory=inventory,
                variable_manager=variable_manager,
                loader=loader,
                options=options,
                passwords=passwords,
                stdout_callback=results_callback)
            result = tqm.run(play)

            resp.status = falcon.HTTP_201
            resp.body = json.dumps({'status': 'success', 'inventory': 'Added {0} to inventory'.format(host)})
        finally:
            if tqm is not None:
                tqm.cleanup()


app = falcon.API()
horde = HordeCollector()

app.add_route('/', horde)
```
