import sys

ABSOLUTE_ZERO = {"C": -273.15, "F": -459.67, "K": 0.0}

UNIT_NAMES = {"C": "Celsius", "F": "Fahrenheit", "K": "Kelvin"}


def celsius_to_fahrenheit(c):
    return c * 9 / 5 + 32


def celsius_to_kelvin(c):
    return c + 273.15


def fahrenheit_to_celsius(f):
    return (f - 32) * 5 / 9


def fahrenheit_to_kelvin(f):
    return fahrenheit_to_celsius(f) + 273.15


def kelvin_to_celsius(k):
    return k - 273.15


def kelvin_to_fahrenheit(k):
    return celsius_to_fahrenheit(kelvin_to_celsius(k))


CONVERTERS = {
    ("C", "F"): celsius_to_fahrenheit,
    ("C", "K"): celsius_to_kelvin,
    ("F", "C"): fahrenheit_to_celsius,
    ("F", "K"): fahrenheit_to_kelvin,
    ("K", "C"): kelvin_to_celsius,
    ("K", "F"): kelvin_to_fahrenheit,
}


def validate_unit(text):
    unit = text.strip().upper()
    return unit if unit in UNIT_NAMES else None


def validate_temperature(value, unit):
    limit = ABSOLUTE_ZERO[unit]
    if value < limit:
        return False, (
            f"{value} {unit} is below absolute zero "
            f"({limit} {unit}). Temperature cannot go lower than that."
        )
    return True, ""


def parse_number(text):
    try:
        number = float(text.strip())
    except ValueError:
        return None
    if number != number or number in (float("inf"), float("-inf")):
        return None
    return number


def convert(value, from_unit, to_unit):
    if from_unit == to_unit:
        return value
    return CONVERTERS[(from_unit, to_unit)](value)


def unit_label(unit):
    return "K" if unit == "K" else f"°{unit}"


def format_result(value, from_unit, result, to_unit):
    return (
        f"{value:,.2f} {unit_label(from_unit)} = "
        f"{result:,.2f} {unit_label(to_unit)}"
    )


def ask_unit(prompt):
    while True:
        unit = validate_unit(input(prompt))
        if unit:
            return unit
        print("  Invalid unit. Please enter C, F or K.")


def ask_temperature(unit):
    while True:
        value = parse_number(input(f"Enter temperature in {UNIT_NAMES[unit]}: "))
        if value is None:
            print("  Invalid input. Please enter a number (e.g. 36.6).")
            continue
        ok, message = validate_temperature(value, unit)
        if ok:
            return value
        print(f"  {message}")


def run_interactive():
    print("=" * 44)
    print("        TEMPERATURE CONVERTER")
    print("   Celsius (C) | Fahrenheit (F) | Kelvin (K)")
    print("=" * 44)

    while True:
        from_unit = ask_unit("\nConvert FROM (C/F/K): ")
        to_unit = ask_unit("Convert TO   (C/F/K): ")
        value = ask_temperature(from_unit)

        result = convert(value, from_unit, to_unit)
        print("\n  Result:", format_result(value, from_unit, result, to_unit))

        again = input("\nConvert another? (y/n): ").strip().lower()
        if again != "y":
            print("Goodbye!")
            break


def run_demo():
    samples = [
        (0, "C", "F"),
        (100, "C", "F"),
        (37, "C", "K"),
        (98.6, "F", "C"),
        (32, "F", "K"),
        (300, "K", "C"),
        (0, "K", "F"),
        (-40, "C", "F"),
        (-40, "F", "C"),
        (25.5, "C", "C"),
    ]
    print("SAMPLE OUTPUTS")
    print("-" * 40)
    for value, src, dst in samples:
        print(format_result(value, src, convert(value, src, dst), dst))

    print("\nINVALID INPUT EXAMPLES")
    print("-" * 40)
    for value, unit in [(-300, "C"), (-500, "F"), (-1, "K")]:
        print(validate_temperature(value, unit)[1])
    for text in ["abc", "", "nan", "12,5"]:
        print(f"parse_number({text!r}) -> {parse_number(text)}")


if __name__ == "__main__":
    if len(sys.argv) > 1 and sys.argv[1].lower() == "demo":
        run_demo()
    else:
        run_interactive()
