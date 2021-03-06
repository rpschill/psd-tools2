Usage
=====

:py:mod:`psd_tools.user_api.psd_image` module provides the user-friendly API
to work with PSD files.

Parsing PSD document
--------------------

:py:class:`~psd_tools.user_api.psd_image.PSDImage` represents a PSD file.

Load an image::

    >>> from psd_tools import PSDImage
    >>> psd = PSDImage.load('my_image.psd')

Print the layer structure::

    >>> psd.print_tree()
    <PSDImage: size=100x200, layer_count=2>
      <Group: 'Group 1', layer_count=1, mask=None, visible=1>
        <ShapeLayer: 'Shape 1', size=41x74, x=25, y=24, visible=1, mask=None, effects=None>
      <PixelLayer: 'Background', size=100x200, x=0, y=0, visible=1, mask=None, effects=None>

Read image header::

    >>> psd.header
    PsdHeader(version=1, number_of_channels=3, height=200, width=100, depth=8, color_mode=RGB)

Access its layers::

    >>> psd.layers
    [<Group: 'Group 1', layer_count=1, mask=None, visible=1>,
     <PixelLayer: 'Background', size=100x200, x=0, y=0, visible=1, mask=None, effects=None>]

Get pattern dict::

    >>> psd.patterns
    {'b2fdfd29-de85-11d5-838b-ff55e75fb875': <psd_tools.Pattern: size=265x219 ...>}

All the low-level internal data are kept in the
:py:attr:`~psd_tools.user_api.psd_image.PSDImage.decoded_data` attribute.


Working with Layers
-------------------

Layers can be one of :py:class:`~psd_tools.user_api.psd_image.Group`,
:py:class:`~psd_tools.user_api.psd_image.PixelLayer`,
:py:class:`~psd_tools.user_api.psd_image.ShapeLayer`,
:py:class:`~psd_tools.user_api.psd_image.TypeLayer`,
:py:class:`~psd_tools.user_api.psd_image.AdjustmentLayer`, or
:py:class:`~psd_tools.user_api.psd_image.SmartObjectLayer`.

:py:class:`~psd_tools.user_api.psd_image.Group` has internal
:py:attr:`~psd_tools.user_api.psd_image.Group.layers`::

    >>> group1 = psd.layers[0]
    >>> group1.name
    'Group 1'

    >>> group1.kind
    'group'

    >>> group1.visible
    True

    >>> group1.closed
    False

    >>> group1.opacity
    255

    >>> from psd_tools.constants import BlendMode
    >>> group1.blend_mode == BlendMode.NORMAL
    True

    >>> group1.layers
    [<ShapeLayer: 'Shape 1', size=41x74, x=25, y=24, visible=1, mask=None, effects=None>]

Other layers have similar properties::

    >>> layer = group1.layers[0]
    >>> layer.name
    'Shape 1'

    >>> layer.kind
    'shape'

    >>> layer.bbox
    BBox(x1=40, y1=72, x2=83, y2=134)

    >>> layer.bbox.width, layer.bbox.height
    (43, 62)

    >>> layer.visible, layer.opacity, layer.blend_mode
    (True, 255, u'norm')

    >>> mask = layer.mask
    >>> mask.bbox
    BBox(x1=40, y1=72, x2=83, y2=134)

    >>> layer.clip_layers
    [<PixelLayer: 'Clipped', size=43x62, x=40, y=72, mask=None, visible=1)>, ...]

    >>> layer.effects
    [<GradientOverlay>]

:py:class:`~psd_tools.user_api.psd_image.TypeLayer` has :py:meth:`~psd_tools.user_api.psd_image.TypeLayer.text` attribute::

    >>> layer.text
    'Text inside a text box'

:py:class:`~psd_tools.user_api.psd_image.SmartObjectLayer` has
:py:meth:`~psd_tools.user_api.psd_image.SmartObjectLayer.linked_data` to obtain
:py:class:`~psd_tools.user_api.embedded.Embedded` object::

    >>> embedded = layer.linked_data()

Raw internal data is accessible by ``layer._info`` property.


Exporting data
--------------

Export a single layer::

    >>> layer.as_PIL()
    <PIL.Image.Image image mode=RGBA size=43x62 at ...>

    >>> layer.mask.as_PIL()
    <PIL.Image.Image image mode=L size=43x62 at ...>

    >>> layer_image = layer.as_PIL()
    >>> layer_image.save('layer.png')

Export the merged image::

    >>> merged_image = psd.as_PIL()
    >>> merged_image.save('my_image.png')

The same using Pymaging::

    >>> merged_image = psd.as_pymaging()
    >>> merged_image.save_to_path('my_image.png')
    >>> layer_image = layer.as_pymaging()
    >>> layer_image.save_to_path('layer.png')

Export layer group (experimental)::

    >>> group_image = group2.as_PIL()
    >>> group_image.save('group.png')
