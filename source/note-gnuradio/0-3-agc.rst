AGC自动增益控制
================

这里整理一下GNURadio的自动增益控制是如何实现的。自动增益控制模块是在analog大类里实现，并一共定义了三种自动增益控制: agc，agc2，agc3。agc是最普通的自动增益控制,agc，agc2增加了attack和delay模式。

.. code-block:: cpp

    class ANALOG_API agc_cc
    {
    public:
        /*!
        * Construct a complex value AGC loop implementation object.
        *
        * \param rate the update rate of the loop.
        * \param reference reference value to adjust signal power to.
        * \param gain initial gain value.
        * \param max_gain maximum gain value (0 for unlimited).
        */
        agc_cc(float rate = 1e-4,
            float reference = 1.0,
            float gain = 1.0,
            float max_gain = 0.0)
            : _rate(rate), _reference(reference), _gain(gain), _max_gain(max_gain){};

        virtual ~agc_cc(){};

        float rate() const { return _rate; }
        float reference() const { return _reference; }
        float gain() const { return _gain; }
        float max_gain() const { return _max_gain; }

        void set_rate(float rate) { _rate = rate; }
        void set_reference(float reference) { _reference = reference; }
        void set_gain(float gain) { _gain = gain; }
        void set_max_gain(float max_gain) { _max_gain = max_gain; }

        gr_complex scale(gr_complex input)
        {
            gr_complex output = input * _gain;

            _gain += _rate * (_reference - std::sqrt(output.real() * output.real() +
                                                    output.imag() * output.imag()));
            if (_max_gain > 0.0 && _gain > _max_gain) {
                _gain = _max_gain;
            }
            return output;
        }

        void scaleN(gr_complex output[], const gr_complex input[], unsigned n)
        {
            for (unsigned i = 0; i < n; i++) {
                output[i] = scale(input[i]);
            }
        }

    protected:
        float _rate;      // adjustment rate
        float _reference; // reference value
        float _gain;      // current gain
        float _max_gain;  // max allowable gain
    };
