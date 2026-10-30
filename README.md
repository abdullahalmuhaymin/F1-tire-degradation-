# F1-tire-degradation-
Developed a Python programme to simulate and analyse F1 tyre performance. Uses lap-time data, statistical trend analysis and graphical visualisation to explore changes in performance across a stint and compare tyre compounds.However, this doesn't prove tyre wear because other racing conditions can affect lap times.

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

COMPOUNDS = {
    "Soft": {"wear": 0.12},
    "Medium": {"wear": 0.08},
    "Hard": {"wear": 0.05}
}

all_data = []


def choose_tyre():
    print("\nSELECT TYRE COMPOUND")
    print("1. Soft")
    print("2. Medium")
    print("3. Hard")

    choices = {"1": "Soft", "2": "Medium", "3": "Hard"}

    while True:
        choice = input("Choose compound: ").strip()

        if choice in choices:
            return choices[choice]

        print("Invalid choice. Try again.")


def enter_lap_times():
    tyre = choose_tyre()
    lap_number = 1
    records = []

    print(f"\nEntering lap times for {tyre} tyres.")
    print("Enter 'done' when finished.")

    while True:
        value = input(f"Lap {lap_number} time (seconds): ")

        if value.lower() == "done":
            break

        try:
            lap_time = float(value)

            if lap_time <= 0:
                print("Lap time must be positive.")
                continue

            records.append({
                "Lap": lap_number,
                "Tyre": tyre,
                "Tyre Age": lap_number - 1,
                "Lap Time": lap_time
            })

            lap_number += 1

        except ValueError:
            print("Please enter a valid number.")

    if records:
        all_data.extend(records)
        print(f"\nSaved {len(records)} laps.")

    else:
        print("No lap times were entered.")


def simulate_stint():
    tyre = choose_tyre()

    print("\nSIMULATION SETTINGS")

    try:
        base_time = float(input("Starting lap time (seconds): "))
        laps = int(input("Number of laps: "))
        wear = float(input(
            "Degradation per lap in seconds "
            f"(default {COMPOUNDS[tyre]['wear']}): "
        ) or COMPOUNDS[tyre]["wear"])

        variation = float(input(
            "Random lap variation in seconds (default 0.08): "
        ) or 0.08)

        seed = int(input("Random seed (default 42): ") or 42)

        if base_time <= 0 or laps < 1 or wear < 0 or variation < 0:
            print("Invalid settings.")
            return

    except ValueError:
        print("Please enter valid numerical values.")
        return

    rng = np.random.default_rng(seed)

    records = []

    for lap in range(1, laps + 1):

        tyre_age = lap - 1

        lap_time = (
            base_time
            + wear * tyre_age
            + rng.normal(0, variation)
        )

        records.append({
            "Lap": lap,
            "Tyre": tyre,
            "Tyre Age": tyre_age,
            "Lap Time": round(float(lap_time), 3)
        })

    all_data.extend(records)

    print(f"\n{tyre} stint simulated successfully!")

    for record in records:
        print(
            f"Lap {record['Lap']:2} | "
            f"{record['Lap Time']:.3f} seconds"
        )


def analyse_data():
    if not all_data:
        print("\nNo data available. Enter or simulate lap times first.")
        return

    df = pd.DataFrame(all_data)

    print("\nTYRE PERFORMANCE ANALYSIS")
    print("=" * 45)

    for tyre in ["Soft", "Medium", "Hard"]:

        tyre_data = df[df["Tyre"] == tyre].sort_values("Lap")

        if tyre_data.empty:
            continue

        times = tyre_data["Lap Time"].to_numpy()
        laps = tyre_data["Lap"].to_numpy()

        print(f"\n{tyre.upper()} TYRES")
        print("-" * 30)

        print(f"Total laps: {len(times)}")
        print(f"Average lap: {np.mean(times):.3f} seconds")
        print(f"Fastest lap: {np.min(times):.3f} seconds")
        print(f"Slowest lap: {np.max(times):.3f} seconds")

        if len(times) >= 2:

            # Linear regression estimates the lap-time trend.
            slope, intercept = np.polyfit(laps, times, 1)

            print(
                f"Lap-time trend: {slope:+.4f} seconds per lap"
            )

            if slope > 0.005:
                print("Pattern: Lap times are increasing.")

            elif slope < -0.005:
                print("Pattern: Lap times are decreasing.")

            else:
                print("Pattern: No strong trend detected.")

            print(
                "Note: A positive trend may be consistent with "
                "degradation, but does not prove tyre wear."
            )

        else:
            print("More laps needed to calculate a trend.")


def compare_compounds():
    if not all_data:
        print("\nNo data available.")
        return

    df = pd.DataFrame(all_data)

    print("\nCOMPOUND COMPARISON")
    print("=" * 45)

    summary = df.groupby("Tyre")["Lap Time"].agg(
        ["mean", "min", "max", "count"]
    )

    print(summary.round(3).to_string())

    print(
        "\nRemember: comparisons are only fair when "
        "conditions and stint lengths are comparable."
    )


def generate_graph():
    if not all_data:
        print("\nNo data available to graph.")
        return

    df = pd.DataFrame(all_data)

    plt.figure(figsize=(10, 6))

    for tyre in ["Soft", "Medium", "Hard"]:

        tyre_data = df[df["Tyre"] == tyre].sort_values("Lap")

        if not tyre_data.empty:
            plt.plot(
                tyre_data["Lap"],
                tyre_data["Lap Time"],
                marker="o",
                label=tyre
            )

    plt.title("F1 Tyre Degradation Analysis")
    plt.xlabel("Lap Number")
    plt.ylabel("Lap Time (seconds)")
    plt.legend()
    plt.grid(True)
    plt.tight_layout()

    plt.savefig("tyre_degradation_graph.png", dpi=150)
    plt.show()

    print("Graph saved as tyre_degradation_graph.png")


def export_data():
    if not all_data:
        print("\nNo data available to export.")
        return

    df = pd.DataFrame(all_data)

    df.to_csv("f1_tyre_data.csv", index=False)

    print("\nData exported to f1_tyre_data.csv")


def main():

    print("=" * 45)
    print("       F1 TYRE DEGRADATION LAB")
    print("=" * 45)
    print("Educational motorsport engineering project")

    while True:

        print("\nMAIN MENU")
        print("1. Enter my own lap times")
        print("2. Simulate a tyre stint")
        print("3. Analyse tyre degradation")
        print("4. Compare tyre compounds")
        print("5. Generate performance graph")
        print("6. Export data to CSV")
        print("7. Exit")

        choice = input("\nSelect an option: ").strip()

        if choice == "1":
            enter_lap_times()

        elif choice == "2":
            simulate_stint()

        elif choice == "3":
            analyse_data()

        elif choice == "4":
            compare_compounds()

        elif choice == "5":
            generate_graph()

        elif choice == "6":
            export_data()

        elif choice == "7":
            print("\nThank you for using F1 Tyre Degradation Lab!")
            break

        else:
            print("Invalid option. Please choose 1-7.")


if __name__ == "__main__":
    main()
