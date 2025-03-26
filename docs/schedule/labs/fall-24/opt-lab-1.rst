Optional Lab 1: Implicit Compliance
===================================

Goal
----
Last quarter, Ankush and I worked on a project by replacing the end effector of the Pupper with DenseTact, a highly fine-grained and compliant tactile sensor. We fathom that with the additional tactile feedback, Pupper would be able to receive more information about the terrain when it walks, and thus we can develop a more compliant and dynamic walking pattern for Pupper. Your goal in this lab is to verify if that's a good idea or not.

Setup
-----

Text Highlighting Examples
-------------------------

Here are different ways to highlight text in RST:

* **Bold text** using ``**text**``
* *Italic text* using ``*text*``
* ``Code or literal text`` using `` ``text`` ``
* :emphasis:`Emphasis` using ``:emphasis:`text``
* :strong:`Strong emphasis` using ``:strong:`text``
* :subscript:`subscript` using ``:subscript:`text``
* :superscript:`superscript` using ``:superscript:`text``

You can also use inline code blocks with different languages:

.. code-block:: python
    def example():
        print("This is a code block")

.. code-block:: bash
    $ echo "This is a bash code block"

Math Expressions
---------------

You can include LaTeX math expressions in RST files in two ways:

1. Inline math using ``:math:`E = mc^2` ``: :math:`E = mc^2`

2. Block math using ``.. math::`` directive:

.. math::
    \begin{align}
    F &= ma \\
    E &= mc^2 \\
    \end{align}

3. Numbered equations using ``.. math:: :label:``:

.. math:: :label: eq:newton
    F = ma

You can reference numbered equations like this: :eq:`eq:newton`

