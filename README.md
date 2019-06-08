## OConf specification draft


```
" OConf specification draft. (c)1996-2019 Ohir Ripe. CC BY License.

  Introduction:

    "It is expected that casual reader will understand every example."

    OConf can describe heterogeneous data trees of arbitrary depth.
    It delivers on all unfulfilled promises of YAML using less
    syntax constructs than a half of TOML's.

    OConf syntax was carefully designed to be easy readable by humans
    and to be single-pass parseable with a loop/switch construct (in C).
    Utf8 encoded unicode and 8 bit codepages can be parsed alike.

      - No quoting/escaping is needed neither for keys nor values.
      - Indentation does not matter but is defined for human read.
      - Long values may span many lines and still keep indent line.

    All but one OConf syntax elements are more than less customary:

         :  colon surrounded by spaces is the key/value separator.
        "   whole line comment may start with ", # or / (usually as //).
        //  two slashes after a space make a comment to the end of line.
        >>  Section, a named record, dictionary or list, is the basic
            structure block.


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


    OConf allows also for data annotation with pragmas called 'meta'.
    These can be used for many purposes, eg. as a hint for translator.

      […]. 'meta' contains a string (…) that is accessible after parse.
        %. 'join meta'. Annotation for the value of this line is given
           as value in the next line.


    Pragmas can be chained.  Eg. |\^[xyType]. pragma string means:
    preserve trailing space, unquote escapes then add an endline.
    The 'meta' block pragma likely provides a value's type.

        _  'filler' character (underscore) can be used to visually
            separate joined pragmas. Eg. |_^_+. equals to |^+.


  TL;DR oconf example:

      A key : value                // name:value line. Name is 'A key'.
    Section : >                    // > tells we're going to depth 1
       spaced :  val & spaces   |. // keep blanks intact.
     SubSec : >>                   // >> go depth 2
          key : value              // /Section/SubSect/key = value
      SSSub : >>>                  // >>> go depth 3
          key : value              // /Section/SubSect/SSSub/key = value
    OthSect : >                    // Next sect opens. All above close.
          key : value              // /OthSect/key = value
         long : long string can +. // next line is a continuation.
              :  be chopped to  +. // this too has a continuation.
              :  many pieces us +. // below the value ends but it needs
              : ing a pragma +. '. // to be disambiguated due to the +. 


    Full OConf specifcation adds 'type' pragmas, more 'meta' brackets
    and structure constructs: [list], <set>, (group) and {dictionary}.
    OConf can also be used as a crystal clear source for XML configs:

      label : R&D        <tag id=?+ kind=?">.  // OConf 'oc2xml' source 
      
      <tag id="1" kind="label">R&amp;D</tag> <!-- XML output produced-->




                       FULL OCONF SPECIFICATION



  Vocabulary:

    * OConf stands for 'Object Config' (or ohir's config ;).
    * SOL and EOL means 'start of line' and 'end of line', respectively.
    * ITEM is a line that parsed successfully to a name, value, and
      possibly other OConf syntax tokens.
    * The 'name' and 'value' words in this document almost universally
      mean the name and value out of a parsed ITEM. 
    * The 'named value' or 'unnamed value' phrases are preferred but
      the 'key' for name,  or 'key/value' phrase may be used too.
    * The 'byte', 'char' and 'character' are used interchangeably
      as OConf line format is specified in terms of byte values.
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
    allowed in an input line. 
    
    Except above, all UTF-8 encoded unicode codepoints are allowed
    as well as eight-bit characters of ISO-8859 and DOS codepages.

    For unicode input a valid UTF-8 encoding is expected.

    Leading space (0x20) characters do not matter (are ommited).

    Line ending NL character MUST end any parseable line.
    The CR of CRLF line ending is considered to be a space character. 
    

  Comments:

    Any line where starting character is one of ! " # / set is an
    explicit comment line. End-of-line comments (aka remarks)
    are introduced with customary // double slash lead.
    Implementations MAY support unmarked comment lines too.
    See [Comment lines] section.
    

  Config line:

    Every OConf line has four, possibly empty, parts. Colon character
    is a separator that MUST be present between the name part and the
    value part.  Every config line provides some VALUE, possibly empty,
    under optional NAME. This pair is hereafter referred to as "ITEM".
    The FLOW and REM parts are auxilary.
    
    cfline ::= (NAME|INDEX ' ')? : (' ' VALUE)? (' ' FLOW)? (' '//REM)?
      ITEM ::= (NAME?|INDEX?) : VALUE?

    OConf line syntax in w3c EBNF:

        TODO


    OConf line syntax in prose: 

    Start Of Line
        (skipped:  any amount of space characters by the input rule)
        optional: <NAME> or <INDEX> followed by a space, then
        required: colon,  then
        optional: <VALUE> prepended with space or colon, then
        optional: <FLOW>  prepended with space, then
        optional: <REM>   prepended with space, slash, slash; then
    End Of Line


    1. NAME (key) MUST start with a byte over 0x2F or with a quote (')
       character that is then skipped. Name characters are not further
       restricted but in MDC context names SHOULD NOT contain a 0x2F
       ('/' slash) character. If it is allowed by an implementation,
       in iall references slash should be substituted. Eg. with tilde.

       Name that consist only of ascii digits [0-9] is called an INDEX.

    2. VALUE is not restricted and needs no enclosing quotes.

    3. FLOW part is described in [FLOW pragmas] section.

    4. REM part opens with (space, slash, slash) sequence and extends
       to the end of line, including all spaces.  Leading space is a
       part of the opening pattern!  Ie. VALUE of 'http://example.tld'
       needs no disambiguation.

       Remark part MUST NOT contain a sequence that resembles pragma.
       Ie. it MUST NOT contain sequences of (punctuation, dot) shape.
    
    5. ITEM forms are hereafter referred to as named or ORD:

        NAME : VALUE  //      Named value.
             : VALUE  // ORD. Ordered value. Name is empty.
       INDEX : VALUE  // ORD. Ordered with explicit index position.


  OConf's data structure mappings:

    Nine values are special and each opens or closes the structure
    block. If these need to be in a VALUE they must be disambiguated.
    
    SECT is the basic block that maps to a data tree, usually a
    dictionary or struct.  It opens with a named VALUE of one or more >
    characters.  Count of > chars describes the depth of the tree.
    SECT closes either at the end of file or where an other SECT
    of the same or of shallower depth level opens.

       name : >   // SECT. A 'name' named section opens at depth 1.
        sub : >>  // count of >s tells the section's depth. Here 2.
         d2 : >>  // SECT of any given depth closes where other opens on
         d1 : >   // the same or shallower depth; or at the end of file.


    Four other structure blocks are recognized. Each opens with a VALUE
    of a single bracket character and closes with a bracket to the pair.

            : [   // LIST. Of ordered values.    Closes with unnamed ']' 
            : {   // DICT. Aka TREE or STRUCT.   Closes with unnamed '}'
            : <   // SET. Implementation dep.    Closes with unnamed '>'
            : (   // GROUP is for common pragma. Closes with unnamed ')'

    LIST, DICT and possibly SET are commonly referred as NEST blocks.
    SET is recognized but its meaning is implementation dependent.
    Opening of any block can either be named or ordered but its
    closing MUST NOT be named.

    
  FLOW pragmas:

    Where other formats use complicated syntax rules, OConf uses short
    after-value pragmas. In most implementations up to six.

    FLOW block begins with a single leading space followed by a symbol
    or punctuation character and always ends with a dot.

    Every pragma is given as a single 'pragma character':

     char  name  meaning
     ‾‾‾‾ ‾‾‾‾‾  ‾‾‾‾‾‾‾   
        '  disa: disambiguate. Used where a VALUE might be confuding.
           - if VALUE contains a (space, slash, slash) sequence. ' //'
           - if VALUE contains only > characters.                '>>>'
           - if VALUE contains a single ( { [ < ] } ) bracket.     '<'
           - if VALUE ends with any pragma sequence.             ' +.'
           - if human decided so.

        | guard: like ' disa plus preserve trailing space of the VALUE.
                 Only one of disambiguation pragmas may appear at line.

        ` btick: value is to be further parsed or modified in some way.
                 Ie. it may instruct to substitute surrounding `s of the
                 VALUE with spaces. "````val``" => "    val  ".

        \ unesc: unescape VALUE. Process common escapes like \t or \n.
                 Also: suppress default escaping of the VALUE (oc2xml).

        ^ nline: add a trailing newline to the string VALUE before use.
                 ^ MAY repeat consecutively adding more newlines. 
                 ^ MUST NOT be used with type pragmas (^ is implicit ")

        ≡  type: ≡ stands for one of:
            22   "   STR : just a string. Also "no-magic-here" switch.
            3f   ?  BOOL : 0, empty, first char Nn or Ff means FALSE.
            23   #   INT : unsigned integer, integer, number.
            24   $   DEC : decimal number (fixed point, currency).
            2c   , CSVAL : VALUE contains a csv record.
            2d   - HYPHE : custom type. Usually SIGNED integer. 
            7e   ~ TILDE : custom type. Usually FLOAT.
            2a   * ASTER : custom type.

                 Explicit typing is discouraged in human-editable
                 config files. Consider 'Model Default' typing instead.

                 Any type MAY convey other than described meanings if it
                 is needed or just useful for particular implementation,
                 but then such meaning MUST be documented and it SHOULD
                 be documented in a config file too (in a comment line). 

        +  join: VALUE with next line value.
        %  join: META. Implementation specific meta is in the next line.

                 Only one of join pragmas may appear in a line.
                 Line to join may have its own pragmas that are applied
                 before VALUEs are join or used. 

        _  fill: underscore is a filler with no other function than to
                 pad for alignment or make longer chains more readable.

       «»  meta: «» pair stand for one of recognized pairs:

       =/  =.../ PATH COPY or REFERENCE to an object at ... path.
       @;  @...; MUSER  Implementation defined.
       ()  (...) MUSER  Implementation defined.
       []  [...] MUSER  Implementation defined.
       <>  <...> MTAG   Used with ML converters.
       {}  {...} MODEL  Used with ML converters.
                 
                 Only one kind of meta SHOULD appear in a line.
                 PATH parts must be valid names or indices.
                 PATH meta usually is used with structure,
                      ie. after [ { < or > special values.

                 Use of a meta in pragma and %+ joins does not make
                 much sense but can possibly be used by some
                 implementations so it is just discouraged.

        .   end: Dot ends the FLOW block.

    Simplified OConf uses only six pragmas: ' | \ ^ + % and a […] meta.
    It is called 'Basic' then.

    Pragmas can be coupled, if more than one should apply.
    Coupled pragmas SHOULD appear in an exact order:

      disa/guard:   ' |                     (one of)
          modify:   ` \ ^
            type:   " ? # $ , ~ * - _       (one of) 
            join:   + %                     (one of)
            meta:   @…; =…/ (…) […] {…} <…> (one of pairs)
    

    Eg.    xy ^^+.  // add two newlines then append next.
           xy  |+.  // preserve spaces then append next.
         ``xy` `+.  // subst ` with space then append next.
         xy +. '+.  // disambiguate +. of the value then append next.


    If pragmas are chained, the 'disa or |guard pragmas MUST always come
    first and any 'meta' block MUST come last.


  GROUP items for common pragma.

    GROUP opens with a VALUE of single parenthesis character. 
    It closes over a block of ITEMs but it does not change its depth
    (so GROUP can not be a part of the path.)
    
    Pragmas join, type, newline, backtick and meta can be set on a GROUP
    opening and these then applly to EACH of grouped ITEMs. Pragmas from
    the GROUP and a line are joined.

        : ( ^+.                     // group multiline text
          : line 1                  // ^+. implicit
          :  line // to disa '.     // '^+. sum of pragmas applies.
          :  line 3                 // + here will break at ': )'
        : )


  Order of appearance for structured data blocks:

    1. At the root of the tree is an unnamed section of depth 0.

    2. All simple values of the section SHOULD be placed
       before the first NEST block ('compound after simple' rule).
    
    3. All values of the section MUST be placed before the first
       subsection lead (SECT of higher depth number).

    4. subsections depth MUST always step down only by one.
       Ie. > then >> then >>> ...

    5. Ordered values take the index number from their consecutive
       place of appearance within its LIST block or from an explicit
       INDEX number otherwise.  An implementation MUST NOT rely on
       the order of explicit INDEXes given.

    6. Any data (subtree) to copy (=/) or reference (=&/) MUST exist or
       at least be defined before it is copied or referred to. 

    7. An implementation MUST NOT assume ordering other than set
       in above rules. It SHOULD report wrong order to the user.
    

    Example:

       aSECTion : >
            a_key : named simple values should come first
            b_key : in some order, eg. lexical one.
                    " NEST collections may come second.
           a_nest : [ 
                    :       with naturally ORDered values, or with
                  8 :       explicit INDEXed values that need not to
                  7 : {     be given in the natural order.
                 some : value 
                    : }
                  : ]
                  1 :       But MUST NOT overwrite any existing index.
                  " subsections MUST come last.
       AsubSECT : >>            // can be empty 
       BsubSECT : >>            // but MUST come (even empty) in depth
         SubSub : >>>           // order, if there is anything
        SuSuSub : >>>>          // to be set down the tree
           ssskey : sssvalue    
       CsubSECT : >>            // BsubSECT whole tree closes here
           a_skey : value       // now we are in /aSECTion/CsubSECT
       bSECTion : >             // aSECTion closes, bSECTion opens.

        
  PATH:
    
    Any node in the OConf data tree can be visualised and pointed to
    using PATH notation where names and indices are concatenated by
    a slash character.  Eg.  /aSECTion/BsubSECT/SubSub/SuSuSub/ and
    /aSECTion/a_nest/7/some/ are valid paths of above example.
    Former points at the subtree, later to the value.

    In MDC implementations common model names usually start with @
    character, then meta path to copy is given short: =@modelnamer/.


  Comment lines:

    OConf comment lines are 'top' ones. Group of consecutive comment
    lines always belongs to the next valid ITEM line. Except for the
    start/end of file positions that should keep their adjacent
    comment lines bond.

      "   Double-quote is the default OConf comment line marker.
      /   Slash is the secondary OConf comment line marker.
          Usually doubled to be consistent with EOL remarks.
      !   Bang is used for in-place error reporting. Usually doubled.
      #   Sharp can be used too.

    If an implementation supports comments at all, it MUST recognize
    above four markers.
    
    If an implementation is expected to write back into config files,
    it SHOULD use error-comments as a reporting venue. Eg.

        Section  : >>>
          !! Error : Unrecognized "someinvalidkey"
          !!         someinvalidkey : somevalue
          keyvalid : valuevalid

    An empty line and a line with only space characters SHOULD be
    treated as a comment line too.

    An implementation MAY allow for free (unmarked) comments where
    every config line that lacks separator is treated as a comment.


  Line pragmas:
    
    If the starting character is from the 0x24-0x26 then 0x28-0x2E
    ranges, line MAY be interpreted as a 'line-pragma'.

    Defined line pragmas (meaning) are:

        . include file
        - raw line (embed some other format)
        + object separator or modifier.
        % define some macro, hint or template


    Nonetheless, line pragmas are just a comment lines and any meaning
    of them (their content) is external to this specification.


  ITEM modifiers:
    
    If the starting character is from the described below set 
    it MAY affect the ITEM given further in the line:

        ' NameStart. Name starts at the next position. Eg with a space.
          ' mod SHOULD be implemented. 
          
    Below three modifiers are just RECOMMENDED, as external to the line
    format specification. (Put here as a reminder.)

        = ReadOnly ITEM. Even app may not modify it.
        > SetOnce. Further overwrite from the config is prohibited.
        < MustBeSet. Human should overwrite default or at least ack it.


  Output indentation rules.

    This section normalizes machine output of the OConf format.
    An implementation that does write OConf files meant for
    human editing MUST abide to it.

    Lengths and positions are measured in terminal units (cells), not
    bytes or glyphs so the indentation keeps even if user is mixing
    ascii (as she must) and her native (wide) script glyphs.
    Ie. colons of the structure block SHOULD keep line even for
    keys given as mix of ascii, EE, Chinese and Devanagari glyphs.

    Implementation MAY narrow wide-print recognizing to the whole
    script ranges. Ie. a rarely used wide-print glyphs may not be
    recognized and may distort the output. 


    The depth of the data tree starts with 0, for values given at root.
    The maximum length of the name is a KMAX parameter to the Printer.

    1. Separator of the SECT opening, regardless of its depth, MUST be
       placed at column equal to the KMAX + 1.  At the same column are
       placed all separators of depth 0.

    2. The colon of the separator used inside any structure block
       MUST be placed at the very same column as the opening/closing
       special value of this block: ( { [ { < or first > of the SECT.

    3. The name (key) of the SECT opening MUST start at column equal to
       depth + 1 if key length is less than KMAX-depth. Or at the first
       column otherwise. Ie. SECT name MUST appear shifted toward SOL.

    4. The end character of the name (key) of any other ITEM line MUST
       be adjacent to the separator. Ie. it MUST appear end-adjusted to
       the following colon (then to the vertical line made of colons).

    5. Value MUST start immediately after separator's trailing space.

    6. If only a few keys of many are expected to be significantly
       longer than others, KMAX then MAY be set lower and these
       outstanding keys' separator MAY not keep to the line.

    7. Colon column for the line can be computed as follows:

        ROOT   any: fcc = KMAX + 2 
        SECT  open: occ = KMAX + 2
        SECT items: scc = KMAX + 4
        NEST items: ncc = scc + (2 * relative_depth).

        where relative_depth is the absolute (path) depth
        minus depth of the current SECT.
        
    All examples in this document use defined above formatting.


  Parsing errors:

    If a config line can not be succesfully parsed it is an error.
    It SHOULD be somewhat reported to the user.  RECOMMENDED message
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
    OConf comment-line markers may claim to be conformant.

    Conformant parser MUST NOT redefine meaning of any ommited part.

    Conformant parser MAY set size limits.


  History:

    Format now named OConf was born in late nineties because none
    other then known allowed for unaltered perl regexp or code
    snippet as a value.  To my (ohir's) knowledge no other config
    format can make to it in a single line even today.

    First OConf parser was implemented as simple regexp for a huge
    mainframe-to-unix pipeline written in perl. Now, shortened with
    new perlre lookaheads and extended to cover also corner cases,
    this parser's tokenizer was used as a syntax formal specification.

    Current OConf specifcation was extended with more metas
    and with definition of GROUP pragma application.




APPENDIX F. 


                            User instructions:

    - Beginning ' of the key always is being stripped. Use two for one. 

    - Space after the VALUE is stripped unless guarded by a |. pragma.
 
    - Disambiguations '. or |. must be used where VALUE:
       ‣ contains a ' //' (remark) sequence.
       ‣ ends with a (pragma, dot) sequence.
       ‣ consist of only > characters (one or more).
       ‣ is a single character of: ( { [ < ] } )
     
    - The single space after the separator colon NEVER is a part of
      the value.  If the leading space in value is significant, second
      colon may be given in place of separator's space to give a visual
      hint:
                There :: is a space before 'is'. // oconf line


      Manual by example: 

        "  Disambiguate      VALUE is:
        key : >>>     '.  // not a Section lead.
            : >       '.  // not an end of SET. Also   ) } ]  
            : [       '.  // not a start of LIST. Also ( { <
            : si ?.   '.  // not 'si' of a bool type

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
