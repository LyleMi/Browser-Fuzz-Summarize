one fuzzing template revealed over 100 IE UAF
==================================================

这篇文章出自black hat Europe 2014的一篇讲稿

作者提出

::

    – Engineers are not good at repairing
    – Engineers make mistakes taking things apart (undoing) 
    – Engineers made mistakes putting things back together (redoing) 

从这里就延伸出去想到工程师可能会犯错的地方，然后fuzzer就从下面这些地方的思路开始构造

- Explicit Pairings
    – Direct: 'on/off', 'true/false', properties.
    - e.g. 

        + ``display="block"/"none"``
        + ``appendChild/removeChild``
        + addEventListener : ``focusin/focusout``
- Implicit Pairings
    - Indirect: inheritance, nullity, state change.
    - e.g.

        + Content: ``innerText=''/ document.write('')``
        + Relation: swap parent/child node
        + Status: ``window.navigate('') / location.reload()``
- Hybrid Pairings
    - Complexity of mixing explicit and implicit.
    - Script (Dynamic) + HTML (Static)

        + ``<body contentEditable='true'>``
        + ``Document.body.contentEditable='false';``
    - Property + Method
- Pairing Combinations
    – Multiple pairings per page.
