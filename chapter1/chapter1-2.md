### Node js2c源码分析

node.gyp
```
  {   
      'target_name': 'node_js2c',
      'type': 'none',
      'toolsets': ['host'],
      'actions': [
        {   
          'action_name': 'node_js2c',
          'process_outputs_as_sources': 1,
          'inputs': [
            '<@(library_files)',
            './config.gypi',
          ],  
          'outputs': [
            '<(SHARED_INTERMEDIATE_DIR)/node_javascript.cc',
          ],  
          'conditions': [
            [ 'node_use_dtrace=="false" and node_use_etw=="false"', {
              'inputs': [ 'src/notrace_macros.py' ]
            }], 
            ['node_use_lttng=="false"', {
              'inputs': [ 'src/nolttng_macros.py' ]
            }], 
            [ 'node_use_perfctr=="false"', {
              'inputs': [ 'src/noperfctr_macros.py' ]
            }]  
          ],  
          'action': [
            'python',
            'tools/js2c.py',
            '<@(_outputs)',
            '<@(_inputs)',
          ],  
        },  
      ],  
    }
```
