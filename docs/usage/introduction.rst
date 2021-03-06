Quick introduction
==================

ImagineInterface_ (``Imagine\Image\ImagineInterface``) and its implementations is the main entry point into Imagine. You may think of it as a factory for ``Imagine\Image\ImageInterface`` as it is responsible for creating and opening instances of it and also for instantiating ``Imagine\Image\FontInterface`` object.

The main piece of image processing functionality is concentrated in the ``ImageInterface`` implementations (one per driver - e.g. ``Imagick\Image``)

The main idea of Imagine is to avoid driver specific methods spill outside of this class and couple of other internal interfaces (``Draw\DrawerInterface``), so that the filters and any other image manipulations can operate on ``ImageInterface`` through its public API.

Installation
------------

Phar file (recommended)
+++++++++++++++++++++++

`Download Imagine PHAR file here <https://github.com/avalanche123/Imagine/raw/master/imagine.phar>`_

.. code-block:: php

   <?php

   require_once 'phar://imagine.phar';

   var_dump(interface_exists('Imagine\Image\ImageInterface'));

PEAR package
++++++++++++

Install using pear package:

.. code-block:: console

   pear channel-discover pear.avalanche123.com
   pear install avalanche123/Imagine-beta

Clone from GitHub
+++++++++++++++++

Clone Imagine git repository:

.. code-block:: console

   git clone git://github.com/avalanche123/Imagine.git

then require files as usual

.. NOTE::
   when using git clone or pear install methods, classes don't get registered with autoload and you have to do it yourself, this will change in future.

Basic usage
-----------

Open Existing Images
++++++++++++++++++++

To open an existing image, all you need is to instantiate an image factory and invoke ``ImagineInterface::open()`` with ``$path`` to image as the  argument

.. code-block:: php

   <?php

   $imagine = new Imagine\Gd\Imagine();
   // or
   $imagine = new Imagine\Imagick\Imagine();
   
   $image = $imagine->open('/path/to/image.jpg');

.. TIP::
   Read more about ImagineInterface_

The ``ImagineInterface::open()`` method may throw one of the following exceptions:

* ``Imagine\Exception\InvalidArgumentException``
* ``Imagine\Exception\RuntimeException``

.. TIP::
   Read more about exceptions_

Now that you've opened an image, you can perform manipulations on it:

.. code-block:: php

   <?php

   use Imagine\Image\Box;
   use Imagine\Image\Point;
   
   $image->resize(new Box(15, 25))
      ->rotate(45)
      ->crop(new Point(0, 0), new Box(45, 45))
      ->save('/path/to/new/image.jpg');

.. TIP::
   Read more about ImageInterface_
   Read more about coordinates_

Create New Images
+++++++++++++++++

Imagine also lets you create new, empty images. The following example creates an empty image of width 400px and height 300px:

.. code-block:: php

   <?php

   $size  = new Imagine\Image\Box(400, 300);
   $image = $imagine->create($size);

You can optionally specify the fill color for the new image, which defaults to opaque white. The following example creates a new image with a fully-transparent black background:

.. code-block:: php

   <?php

   $size  = new Imagine\Image\Box(400, 300);
   $color = new Imagine\Image\Color('000', 100);
   $image = $imagine->create($size, $color);

Color Class
+++++++++++

Color is a class in Imagine, which takes two arguments in its constructor: the RGB color code and a transparency percentage. The following examples are equivalent ways of defining a fully-transparent white color.

.. code-block:: php

   <?php

   $white = new Imagine\Image\Color('fff', 100);
   $white = new Imagine\Image\Color('ffffff', 100);
   $white = new Imagine\Image\Color('#fff', 100);
   $white = new Imagine\Image\Color('#ffffff', 100);
   $white = new Imagine\Image\Color(0xFFFFFF, 100);
   $white = new Imagine\Image\Color(array(255, 255, 255), 100);

After you have instantiated a color, you can easily get its Red, Green, Blue and Alpha (transparency) values:

.. code-block:: php

   <?php

   var_dump(array(
      'R' => $white->getRed(),
      'G' => $white->getGreen(),
      'B' => $white->getBlue(),
      'A' => $white->getAlpha()
   ));

Advanced Examples
-----------------

An Image Collage
++++++++++++++++

Assume we were given the not-so-easy task of creating a four-by-four collage of 16 student portraits for a school yearbook.  Each photo is 30x40 px and we need four rows and columns in our collage, so the final product will be 120x160 px.

Here is how we would approach this problem with Imagine.

.. code-block:: php

   <?php

   use Imagine;
   
   // make an empty image (canvas) 120x160px
   $collage = $imagine->create(new Imagine\Image\Box(120, 160));
   
   // starting coordinates (in pixels) for inserting the first image
   $x = 0;
   $y = 0;
   
   foreach (glob('/path/to/people/photos/*.jpg') as $path) {
      // open photo
      $photo = $imagine->open($path);
      
      // paste photo at current position
      $collage->paste($photo, new Imagine\Image\Point($x, $y));
      
      // move position by 30px to the right
      $x += 30;
      
      if ($x >= 120) {
         // we reached the right border of our collage, so advance to the
         // next row and reset our column to the left.
         $y += 40;
         $x = 0;
      }
      
      if ($y >= 160) {
         break; // done
      }
   }
   
   $collage->save('/path/to/collage.jpg');

Image Reflection Filter
+++++++++++++++++++++++

.. code-block:: php

   <?php

   class ReflectionFilter implements Imagine\Filter\FilterInterface
   {
       private $imagine;

       public function __construct(Imagine\Image\ImagineInterface $imagine)
       {
           $this->imagine = $imagine;
       }
    
       public function apply(Imagine\Image\ImageInterface $image)
       {
           $size       = $image->getSize();
           $canvas     = new Imagine\Image\Box($size->getWidth(), $size->getHeight() * 2);
           $reflection = $image->copy()
               ->flipVertically()
               ->applyMask($this->getTransparencyMask($size))
           ;

           return $this->imagine->create($canvas, new Imagine\Image\Color('fff', 100))
               ->paste($image, new Imagine\Image\Point(0, 0))
               ->paste($reflection, new Imagine\Image\Point(0, $size->getHeight()));
       }

       private function getTransparencyMask(Imagine\Image\BoxInterface $size)
       {
           $white = new Imagine\Image\Color('fff');
           $fill  = new Imagine\Image\Fill\Gradient\Vertical(
               $size->getHeight(),
               $white->darken(127),
               $white
           );

           return $this->imagine->create($size)
               ->fill($fill)
           ;
       }
   }

   $imagine = new Imagine\Gd\Imagine();
   $filter  = new ReflectionFilter($imagine);
   
   $filter->apply($imagine->open('/path/to/image/to/reflect.png'))
      ->save('/path/to/processed/image.png')
   ;

.. TIP::
   For step by step explanation of the above code `see Reflection section of Introduction to Imagine <http://speakerdeck.com/u/avalanche123/p/introduction-to-imagine?slide=31>`_

Architecture
------------

The architecture is very flexible, as the filters don't need any processing logic other than calculating the variables based on some settings and invoking the corresponding method, or sequence of methods, on the ``ImageInterface`` implementation.

The ``Transformation`` object is an example of a composite filter, representing a stack or queue of filters, that get applied to an Image upon application of the ``Transformation`` itself.

.. TIP::
   For more information about ``Transformation`` filter `see Transformation section of Introduction to Imagine <http://speakerdeck.com/u/avalanche123/p/introduction-to-imagine?slide=57>`_


.. _ImagineInterface: ../api/image.html#Imagine\\Image\\ImagineInterface
.. _ImageInterface: ../api/image.html#Imagine\\Image\\ImageInterface
.. _coordinates: coordinates.html
.. _exceptions: exceptions.html