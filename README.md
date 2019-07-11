## OConf specification draft, v1.0.0

```
" OCONF specification draft. (c)1996-2019 Ohir Ripe. CC BY License.

  Introduction:

    "It is expected that casual reader will understand every example."

    OCONF can describe heterogeneous data trees of arbitrary depth.
    It delivers on all unfulfilled promises of YAML using less
    syntax constructs than a half of TOML's.

    OCONF syntax was carefully designed to be easy readable by humans
    and to be single-pass parseable with a loop/switch construct (in C).
    Utf8 encoded unicode and 8 bit codepages can be parsed alike.

      - No quoting/escaping is needed neither for keys nor values.
      - Indentation does not matter but is defined for human read.
      - Long values may span many lines and still keep indent line.

    All but one OCONF syntax elements are more than less customary:

        :   colon surrounded by spaces is the key/value separator.
        ^^  Section, a named record, is the basic structure block.
        []  [List], {Dict}, <Set> and (Group) can structure data, too. 
        #"  whole line comment starts with " ! # or / (usually two of).
        //  two slashes after a space make a comment to the end of line.


    The only new syntax element to learn and remember is 'value pragma'
    written as space-punctuation(s)-dot sequence after the value.
    Six pragmas replace all quoting, escaping and complex syntax rules
    other formats impose on parsers and humans alike.

    Oconf 'after-value' pragmas are:
            
        +. 'join' this line value with next line value.
        '. 'disambiguate' value.  Eg. if it contains //.
        |. 'guard' the trailing space otherwise trimmed.
        ^. 'newline' append a newline to the read value.
        \. 'unescape' \t \n and \xHH escapes in the read value.
        `. 'backtick'. Value is of some user defined "special"
            shape; to be processed further by user's code. 


    OCONF allows also for data annotation with pragmas called 'meta'.
    These can have many purposes, eg. as a type hint. Eg.

      {…}. 'meta' contains a string "…" that is accessible after parse.
        %. 'meta' for this line is given in the value of the next line.


    Pragmas can be joined.  Eg. |\^{xyType}. pragma string means:
    preserve trailing space, unquote escapes then add an endline.
    The <meta> block pragma likely hints at value's type.

        _  'filler' character (underscore) can be used to visually
            separate joined pragmas. Eg. |_^_+. equals to |^+.


    Annotated OCONF can be used as a crystal clear source for XML output:

      label : R&D        <tag id=?+ kind=?">.  // OCONF 'oc2xml' source 
      
      <tag id="1" kind="label">R&amp;D</tag> <!-- XML output produced-->


  TL;DR config example with most features shown:
    
// line comment can also begin with " # ! (d-quote, sharp and bang) markers.
 ! free comment lines are possible too - these may not contain ' : ' though.
 # Note that in real configs pragmas and annotations are rare.

 ^ Section : ----- section lead --- //  ^  is a "section" marker and depth indicator. 
 '^ escape : not a section lead     // '   at start makes any key an ordinary string.
    spaced :  val & spaces     |.   //  |  "guard pragma" keeps tail blanks intact.
    noComm : hello // there    '.   //  '  "disa  pragma" makes former // to the Value
   withCTL : Use\t tab and \n  \.   //  \  "unesc pragma" unescapes \t, \n, and \xHH
    withNL : some value        ^.   //  ^  "newline pragma" adds a '\n' at the end.
    looong : value can span    +.   //  +  "join Pragma" joins this line value with
           :  many lines and   +.   //      next line value 
           :: still keep indent.    // ::  separator makes leading space more visible.

  ^^ SubSec : --------------------- // ^^  open subsection at depth 2.
                                    //     ----- in value above is just a decoration
            : list member  0        //     Ordered (unnamed) values can be indexed 
            : list member  1        //     naturally by the order of apperance
        33  : list member 33        //     or with index being given explicit
            : value                 //     /Section/SubSect[34] = value
                                       
   ^^^ SSSub : ----------------  %. // ^^^ go depth 3 sub-sub section. %. next meta.
   'CIenv: testing, triage, bucket  //     meta line for the section as per %. above.
         key : value                //     /Section/SubSect/SSSub.key = value

 ^ OthSect : ---------------------- // Next depth 1 section opens. All above close.
       key : value                  // /OthSect.key = value
     a key : value                  // spaces in keys are ok.   Here is  "a key".
   ' spkey :  value                 // '  quotes leading space. Here is " spkey".
       Имя : Юрий                   // OConf supports utf-8 encoded unicode
       键k : v值                    // in full range. [And 8bit "codepages" too].

 ^^ PGroups : --------------------- // ( Group applies a pragma to many items
    ( : group pragma ^+.            // Put ^+. on every line till group ends.
      : many lines may come here    //  Eglible are metas and pragmas + \ ` ^. 
      :  that keep indent line but  //  | ' % can not be grouped. 
      :  sometimes need to be disa  //
      : mbiguated for // or ?.  '.  // Here sum of pragmas applies: '^+.
      : 
    ) : group ends                  // Bracket can have a value or pragma, too.

 ^^ SubTrees : -------------------- // Show other structure constructs.
  dictname { : dict opens           // These SHOULD NOT be used
        some : value                // in human __editable__ configs.
     'nodict { :                    // 'disa makes key ordinary: "nodict {" 
    listname [ : list opens         // Ordered (unnamed) values can be indexed 
               : list member 0      // naturally by the order of apperance
           33  : list member 33     // or with index being given explicit
               < : anon set opens   // <set> is now at index 34
                 : with unnamed    
             and : with named members 
            '33  : 33 is a string not an index due to opening 'disa
            ''7  : '7 is a two characters string
        deepdict { : dictionary in a set, looked up by its name
              deep : value here  // /OthSect/SubDict/listname/34/deepdict/deep/
                 } : deep dict closes
               > : set closes
               : list member 35
             ] : list closes
       other : value
           } : dict closes

 ^^ Raw Multiline :   // Use :==    
  mtx :== xHereRaw    // below multiline block will be a VALUE of mtx. 
      This^^^^^^^^ is a custom boundary string. It must have at least
      8 bytes and exactly 8 bytes of it matters. If no custom boundary
      given (or it is too short), the ==RawEnd default one is used.
      Here block ends at a space before the x of xHereRaw.





                       FULL OCONF SPECIFICATION


  Vocabulary:

    * OCONF stands for 'Object Config' (or ohir's config).
    * SOL and EOL means 'start of line' and 'end of line', respectively.
    * ITEM is a line that parsed successfully to a name, value, and
      possibly other OCONF syntax tokens.
    * The 'name' and 'value' words in this document almost universally
      mean the name and value out of a parsed ITEM. 
    * The 'named value' or 'unnamed value' phrases are preferred but
      the 'key' for name,  or 'key/value' phrase may be used too.
    * The 'byte', 'char' and 'character' are used interchangeably
      as OCONF line format is specified in terms of byte values.
    * Syntax' punctuation and symbol characters are used without quotes
      where it is unambiguous. Sporadically are given in single quotes.
    * The 'glyph' word is used if unicode representation form matters.
    * Words 'tree' and 'subtree' means some compound and/or heterogenous
      data structure (block).
    * MDC is an acronym for "Model-Default Configuration" (a pattern).

    The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
    "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in
    this document are to be interpreted as described in RFC 2119.


  Input:

    Ascii control characters (0x00-0x1F and 0x7F) SHOULD NOT be
    allowed in an input line, except for TAB (0x09), and CR (0x0d)
    characters that are treated like a space (0x20) and a NL (0x0a)
    character that MUST end any parseable line. Leading space (0x20)
    characters of a line do not matter.
    
    Except above, all UTF-8 encoded unicode codepoints are allowed
    as well as eight-bit characters of ISO-8859 and DOS codepages.

    For unicode input a valid UTF-8 encoding is expected.
    

  Comments:

    Any line where starting character is one of ! " # / set is an
    explicit comment line. End-of-line comments (aka remarks)
    are introduced with customary // double slash lead.
    Implementations MAY support unmarked comment lines too.
    See [Comment lines] section.
    

  Config line:

    Every config line provides some VALUE, possibly empty, under
    optional NAME. This pair is hereafter referred to as "ITEM".
    The FLOW, and REM parts are auxilary.

    Every OCONF line has four, possibly empty, parts. Colon character
    is a separator that MUST be present between the name part and the
    value part.  
    
    cfline ::= (NAME|INDEX|SNAME ' ')? : (' ' VALUE)? (' ' FLOW)? (' '//REM)?
      ITEM ::= (NAME?|INDEX?|SNAME?) : VALUE?

    Line syntax in prose: 

    Start Of Line
        (skipped:  any amount of space characters by the input rule)
        optional: <NAME> or <INDEX>, or <SNAME> followed by a space, then
        required: colon,    then
        optional: <VALUE>   prepended with space or colon, then
        optional: <FLOW>    prepended with space, then
        optional: <REM>     prepended with space, slash, slash; then
    End Of Line
              (Formal line syntax description is given in APPENDIX B.) 


    1. INDEX (numerical key) MUST start with an ascii digit and it MUST
       consist only of ascii digits [0-9].

    2. SNAME (structure marker key) MUST either start with a one or more
       ^ (0x5e) caret characters, or end with one or more bracket
       ([{<>}]) characters that are the structure markers. Other parts
       od the SNAME are used as the NAME.

    3. NAME (ordinary key) is a NAME that is neither INDEX nor SNAME.
       If the first character is a single quote ' (0x27) this character
       is then skipped and any string after it is treated as an ordinary
       NAME. 

       A Name (of any form) MUST NOT contain a sequence of colon-space.
       It SHOULD NOT contain a character that is used by implementation
       as an access path joiner (usually a slash or an underscore).

    4. VALUE is not restricted and needs no enclosing quotes.

    5. FLOW part is described in [FLOW pragmas] section.

    6. REM part opens with (space, slash, slash) sequence and extends
       to the end of line, including all spaces.  Leading space is a
       part of the opening pattern!  Ie. VALUE of 'http://example.tld'
       needs no disambiguation.

       Remark part MUST NOT contain a sequence that resembles pragma.
       Ie. it MUST NOT contain sequences of (punctuation, dot) shape.
    
    ITEM forms can be further referred to as Named or ORD (ordered):

        NAME : VALUE  // Named value.
             : VALUE  // ORD. Ordered value. Name is empty.
       INDEX : VALUE  // IDX. Ordered with explicit index position.


  OCONF Structure:

    Structure is introduced using a SNAME, ie a NAME that starts with
    ^ or @ or one of bracket characters:

    Opening        Closing 
        ^ :            ^ : - SECT  (named Section/next named Section)
        @ :            @ : - SECT  second form  
        ( :            ) : - GROUP (common pragma)
        [ :            ] : - LIST  (of ordered VALUEs)
        { :            } : - DICT  (dictionary, associative array)
        < :            > : - SET   (implementation dependent)
    
    SECT is the basic block that maps its items to a struct, dictionary
    or even to a list.  Section opens with an ITEM that is marked using
    caret opening. An implementation MAY use bracket opening instead.

    Depth of the Section, from the root of depth 0, is given in the
    count of opening characters.

    SECT closes either at the end of file or where an other SECT of
    the same or shallower depth level opens.
    
    Any Section can contain other Sections (subsections) thus forming a
    data tree that starts at common root at the depth 0. Every section
    right under the root has depth 1, its subsections are of depth 2
    and so on.

    //  .                                Root has depth 0
    ^   ├── Section depth1 :             section opens at depth 1
    ^^  │   ├── SubSect.d2 :             count of ^s tells the depth 
    ^^^ │   │   └─ SSSu.d3 :             SECT of any given depth closes
    ^^  │   └── SubNext.d2 :             where other section opens on
    ^   └── TheNextSection :             the same or shallower depth;
    ^^      └── ItsSubs.d2 :             or at the end of file.
    
    ()
    GROUP is used to apply a common pragma to the group of ITEMs.

    [] {} <>
    Three other structure blocks (LIST, DICT and SET) SHOULD not be
    used in configs meant to be edited by a human. 
    Structure block closings MUST NOT be named.


  RAW VALUE

    :== BOUNDARY ... BOUNDARY
    RAW (VALUE) starts at column 1 of a line below and ends at
    a character immediately follwed by the custom or default BOUNDARY
    string.  Custom BOUNDARY string must have at least 8 bytes and only
    8 bytes of it matters.  If custom BOUNDARY string is not given,
    it defaults to ==RawEnd.
    

  FLOW pragmas:

    Where other formats use complicated syntax rules, OCONF uses short
    after-value pragmas. In most implementations up to six.

    FLOW block begins with a single leading space followed by a symbol
    or punctuation character and always ends with a dot.

    Every pragma is given as a single 'pragma character':

     char  name  meaning
     ‾‾‾‾ ‾‾‾‾‾  ‾‾‾‾‾‾‾   
        '  disa: disambiguate. Used where a VALUE might be confuding.
           - if VALUE contains a (space, slash, slash) sequence. ' //'
           - if VALUE ends with any pragma sequence.             ' +.'
           - if human decided so.

        | guard: like ' disa plus preserve trailing space of the VALUE.
                 Only one of disambiguation pragmas may appear at line.

        ` vaspe: value is to be further parsed or modified in some way.

        \ unesc: unescape VALUE. Process common escapes like \t or \n.

        ^ nline: add a trailing newline to the string VALUE before use.
                 ^ MAY repeat consecutively adding more newlines. 
                 ^ MUST NOT be used with type pragmas (it is implicit ")

        +  join: VALUE with next line value.
        %  join: META. Implementation specific meta is in the next line.

                 Only one of join pragmas may appear in a line.
                 Line to join may have its own pragmas that are applied
                 before VALUEs join. 

        _  fill: underscore is a filler with no other function than to
                 pad for alignment or make longer chains more readable.

       «»  meta: «» stands for one of recognized pairs:

       &/  &.../ PATH REFERENCE an object at ... path.
       =/  =.../ PATH COPY FROM an object at ... path.
       @;  @...; MUSER  Implementation defined.
       ()  (...) MUSER  Implementation defined.
       []  [...] MUSER  Implementation defined.
       <>  <...> MTAG   usually used with ML converters.
       {}  {...} MODEL  usually used with ML converters.
                 
                 Only one kind of meta SHOULD appear in a line.
                 PATH parts must be valid names or indices.
                 PATH meta usually is used with structure,
                      ie. after [ { < or > special values.

                 Use of a meta in pragma and %+ joins does not make
                 much sense but can possibly be used by some
                 implementations so it is just discouraged.

        .   end: Dot ends the FLOW block.

       Additional eight characters MAY be recognized as 'type' pragmas.
       (These are called 'type' pragmas for historical reasons only as
       explicit typing is discouraged in human-editable config files.
       For new implementations 'Model Default' typing should be used
       instead). Still these pragmas can be useful eg. for Model
       definitions.

        ≡  type: ≡ stands for one of:
            22   "   STR : just a string. Also "no-magic-here" switch.
            3f   ?  BOOL : 0, empty, first char Nn or Ff means FALSE.
            23   #   INT : unsigned integer, integer, number.
            24   $   DEC : decimal number (fixed point, currency).
            2c   , CSVAL : VALUE contains a csv record.
            2d   - HYPHE : custom type. Usually SIGNED integer. 
            7e   ~ TILDE : custom type. Usually FLOAT.
            2a   * ASTER : custom type.


    Pragmas and metas can be coupled if more than one should apply.
    In such a chain of pragmas disa/guard (if present) MUST come first
    and any metas (if present) MUST come last. Other, ie. modify, type
    and join SHOULD appear in order listed:

          modify:   ` \ ^
            type:   " ? # $ , ~ * - _       (one of) 
            join:   + %                     (one of)
    
    Eg.    xy ^^+.  // add two newlines then append next.
           xy  |+.  // preserve spaces then append next.
         ``xy` `+.  // subst ` with space then append next.
         xy +. '+.  // disambiguate +. of the value then append next.


  GROUP items for common pragma.

    GROUP opens with a SNAME of a single parenthesis character. 
    It closes over a block of ITEMs but it does not change its depth
    (so GROUP can not be a part of the path.)
    
    Any pragma that makes sense can be set on a GROUP opening and it
    then applies to EACH of grouped ITEMs. Pragmas from the GROUP and
    a line are joined.

        ( : ^+.                     // group multiline text
          : line 1                  // ^+. implicit
          :  line // to disa '.     // '^+. sum of pragmas applies.
          :  line 3                 // + here will break at ':)'
        ) :


  Order of appearance:

    1. For any given Section depth, simple named values MUST come first,
       then come all NEST blocks, and subsections MUST come last.
       If any is present.

    2. In their respective kind all ITEMs MUST come in some order,
       eg. a lexical one.
       
    3. Depth of any Subsection MUST always step down only by one.
       Ie. ^ then  ^^ then  ^^^ ...

    4. Ordered values take the index number from their consecutive
       place of appearance within its block or from an explicit
       INDEX number otherwise.  An implementation MUST NOT rely on
       the order of explicit INDEXes given.

    5. An implementation MUST NOT assume ordering other than set
       in above rules and it SHOULD report wrong order to the user.
    

    Example:

     ^ aSECTion :  
          a·key : named simple values should come first
          b·key : in some order, eg. lexical one.
                  " NEST collections may come second.
         a·nest [ : 
                  : with naturally ORDered values, or with
                8 : explicit INDEXed values that need not
                7 { : to be given in the natural order.
               some : value 
                  } :
                1 :       But MUST NOT overwrite any existing index.
                ] :
                " subsections MUST come last.
  ^    AsubSECT :          // can be empty 
  ^^   BsubSECT :          // but MUST come (even empty) in depth
  ^^^    SubSub :          // order, if there is anything
  ^^^^  SuSuSub :          // to be set down the tree
         ssskey : sssvalue 
  ^^   CsubSECT :          // BsubSECT whole tree closes here
         a·skey : value    // now we are in /aSECTion/CsubSECT
  ^    bSECTion :          // aSECTion closes, bSECTion opens.

        
  PATH:
    
    Any node in the OCONF data tree can be visualised and pointed to
    using PATH notation where names and indices are concatenated by
    a slash or underscore character.  Eg. 'aSECTion_a·nest_7_some'
    and '/aSECTion/BsubSECT/SubSub/SuSuSub/' are valid paths of
    above example.  Later points at the subtree, former to the
    value.  In MDC implementations common model names usually are
    of SNAME shape and start with @ character. Then meta path on
    a copy is given short: =@modelnamer/.


  Comment lines:

    OCONF comment lines are 'top' ones. Group of consecutive comment
    lines always belongs to the next valid ITEM line. Except for the
    start/end of file positions that should keep their adjacent
    comment lines bond.

      "   Double-quote is the default OCONF comment line marker.
      /   Slash is the secondary OCONF comment line marker.
          Usually doubled to be consistent with EOL remarks.
      !   Bang is used for in-place error reporting. Usually doubled.
      #   Sharp can be used too.

    If an implementation supports comments at all, it MUST recognize
    above four markers.
    
    If an implementation is expected to write back into config files,
    it SHOULD use error-comments as a reporting venue. Eg.

        ^^ Section : 
          !! Error : Unrecognized "someinvalidkey"
          !!         someinvalidkey : somevalue
          keyvalid : valuevalid

    An empty line and a line with only space characters SHOULD be
    treated as a comment line too.

    An implementation MAY allow for free (unmarked) comments where
    every config line that lacks separator is treated as a comment.


  Line pragmas:
    
    If the starting character is from the 0x24-0x26 then 0x28-0x2E
    ranges, this line MAY be interpreted as a 'line-pragma'.
    SUGGESTED meanings for common line-pragmas are:

        . include file
        - raw line (embed some other format)
        + object separator or modifier.
        % define some macro, hint or template


    Nonetheless, line pragmas from the format PoV are just a comments
    and any meaning of them (their content) is external to this
    specification.


  NAME ITEM modifiers:
    
    If a NAME starts with a character that is not an ascii letter or
    digit this and a following character MAY be used in MDC context
    to convey more information. It can be interpreted and impose usage
    rules.  SUGGESTED meanings for some NAME modifiers are:

        == ReadOnly ITEM. Even app may not modify it.
        => SetOnce. Further overwrite from the config is prohibited.
        =< MustBeSet. Human should overwrite default or at least ack it.
        =? Default ITEM, that can be modified by other includes.


  Output indentation rules.

    This section normalizes machine output of the OCONF format.
    An implementation that does write OCONF files meant for
    human editing MUST abide to it.

    Lengths and positions are measured in terminal units (cells), not
    bytes or glyphs so the indentation keeps even if user is mixing
    ascii (as she must) and her native (wide) script glyphs.
    Ie. colons of the structure block SHOULD keep line even for
    keys given as mix of ascii, EE, Chinese and Devanagari glyphs.

    Implementation MAY narrow wide-print recognizing to the whole
    script ranges. Ie. a rarely used wide-print glyphs MAY not be
    recognized and MAY distort the output.

    The depth of the data tree starts with 0, for values given at root.
    OCONF's formatter (printer) is given a single KMAX parameter
    that denotes maximum expected length of any ITEM Name.

    If only a few keys of many are expected to be significantly longer
    than others, KMAX then MAY be set lower and these outstanding keys'
    separator MAY not keep to the line.

    ITEMs within any Section immediate scope have their colon-separator
    placed at column KMAX+2.
    
    ITEMs within any other structure block have their closing bracket
    placed at the column of the colon-separator of the containing
    structure.

    NAMEs and INDEXes are space-adjusted toward the separator.
    SECT ^ markers always start at column 1 but other parts of the
      NAME are shifted toward the separator.

    All examples in this document use defined above formatting.


  Parsing errors:

    If a config line can not be succesfully parsed it is an error.
    It MUST be somewhat reported to the user.  RECOMMENDED message
    is: 'ERROR: line <n> is not valid.'


  Structure errors:

    The + and % join pragmas expect next line to not have a name. 
    If there is a name part in the next line it is an error.
    RECOMMENDED message is: 'ERROR: continuation line may not be named'.

    If two sections resolve to the same path it MAY be an error.
    If it is an error it SHOULD be somewhat reported to the user.
    RECOMMENDED message is: 'ERROR: section {name} repeated at {path}'.

    If named or ordered values at the same path supersede each other,
    the behaviour is implementation dependent but it MAY be an error. 
    If it is an error it SHOULD be somewhat reported to the user.
    RECOMMENDED message is: 'ERROR: unexpected overwrite of: {path}'.

    More errors MAY need reporting, depending on the scope and purpose
    of the implementation. These errors are 'implementation dependent'. 


  Conformance:

    Conformant parser SHOULD implement ONLY parts of this
    specification that are relevant to a particular use.

    In the edge case of an entirely flat config a parser that uses
    just ' : ' key/value separator and skip lines that start with
    OCONF comment-line markers may claim to be conformant.

    Conformant parser MUST NOT redefine meaning of any ommited part.

    Conformant parser MAY set size limits.


  History:

    Format now named OCONF was born in late nineties because none
    other then known allowed for unaltered perl regexp or code
    snippet as a value.  To my (ohir's) knowledge no other config
    format can make to it in a single line even today.

    Current OCONF specifcation was extended with more metas and with
    definition of a GROUP for pragma application. Structure markers
    (brackets) in the first implementation were special values, now
    these were moved to the NAME. 
    
    This reworked specifcation is tagged as v1.0.0





APPENDIX F. 


                            User instructions:

    - Beginning ' of the key always is being stripped. Use two for one. 

    - Space after the VALUE is stripped unless guarded by a |. pragma.
 
    - Disambiguations '. or |. must be used where VALUE:
       ‣ contains a ' //' (remark) sequence.
       ‣ ends with a (pragma, dot) sequence.
     
    - The single space after the separator colon NEVER is a part of
      the value.  If the leading space in value is significant, second
      colon may be given in place of separator's space to give a visual
      hint:
                There :: is a space before 'is'. // oconf line


      Manual by example: 

        #  Disambiguate          result shown in "double-quotes": 
        key : va //lue '. // disa remark - gives "va //lue" string
        key : value +. '. // disa pragma - gives "value +." string
        ' !#?%key : value // disa kstart - gives " !#?%key" string
        '@__  key :       // disa kstart - @__ is not a Section lead.

        "  Trailing guard 
        key : value       // 'value'     - NO trailing spaces
        key : val //ue '. // 'val //ue'  - NO trailing spaces
        key : val //ue |. // 'val //ue ' - ONE trailing space
        key :  value      // ' value'    - ONE leading space
        key :: value      // ' value'    - with a :: visual hint
        key : value |.    // 'value '    - ONE trailing space
        key :: value |.   // ' value '   - space on both sides
        key :  value |.   // ' value '     with and without ::
        key :  |.         // ' '         - SINGLE space
        key :: |.         // ' '         - SINGLE space

        "  Join and spaces
        key : value  +.   // 'value' +  
        key : vcont       // 'vcont'    :gives 'valuevcont'  
        key : value  +.   // 'value' +  
        key :: vcont      // ' vcont'   :gives 'value vcont'
        key : value  +.   // 'value' +  
        key :  vcont      // ' vcont'   :gives 'value vcont'
        key : value |+.   // 'value ' +  
        key : vcont       // 'vcont'    :gives 'value vcont'

        "  Empty values
        key :             // '' EMPTY preferred
        key : |.          // '' EMPTY discouraged
        key :     '.      // '' EMPTY discouraged 

        "  Separator cheatsheet
        key : value  // (space, colon, space) is the default
        key :        // (space, colon, EOL) is for an empty VALUE
            : value  // (colon, space) is the lead of an ordered VALUE
            :        // (colon, EOL) gives an empty ordered VALUE
        key :: value // doubled colon emphasizes leading space
            :: value // 


    OConf parsers are byte oriented. In a righ-to-left (RTL) language
    environment all bracket characters in special values still MUST be
    given as ascii.
                           'aleph, beth, gimmel' dict opens // { : אבג
```
