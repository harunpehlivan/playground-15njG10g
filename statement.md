# Simulated Annealing in C#

In this example, we will doing a simple thing : adjusting one coefficent for having a better results for the algorithm.
![Image for Simulated Annealing](https://github.com/iadevoops/playground-15njG10g/blob/master/sa.gif)


```C# runnable
// { autofold
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SimulatedAnnealing
{
    class Program
    {
        static Random rand = new Random();
        static int ct0, ct1, ct2, ct3;

        static double random_x(double xmin, double xmax)
        {
            return xmin + (xmax - xmin) * rand.NextDouble();
        }

        static double random_variate(double x, double xmin, double xmax, double delta)
        {
            var x0 = x - delta < xmin ? xmin : x - delta;
            var x1 = x + delta > xmax ? xmax : x + delta;

            return random_x(x0, x1);
        }

        static double energy(double x)
        {
            return COEF * myFunction(x);
        }

        static double acceptanceProbability(double energy, double new_energy, double temp)
        {
            if (new_energy < energy)
            {
                ct1++;
                return 1.0;
            }

            return Math.Exp((energy - new_energy) / temp);
        }

        static int COEF = 100;
        static double simulated_annealing()
        {
            double XMIN = -5;
            double XMAX = 5;

            double Temperature = 1000;
            double minTemp = 1;

            // We change this coef for adjusting the decreasing of temperature. In this case, for doing 200 simulations. 
            //coolingrate = -log(minTemp/Temperature) / nbSimulation
            double coolingRate = 0.034;

            double searcheGlobalMinimum = random_x(XMIN, XMAX);
            double searchLocalMinimum = random_x(XMIN, XMAX);
            double currentValue = random_x(XMIN, XMAX);

            ct0 = ct1 = ct2 = ct3 = 0;
            while (Temperature > minTemp)
            {
                // The variation of delta for a new value is not in the real Simulated Annealing algorithm .
                // In range [-5,5], a jump +-2 is correct for searching a far solution and +-0.1 for searching a near solution.
                //double delta = rand.Next(10) == 0 ? 2 : 0.1; 
                //currentValue = random_variate(searchLocalMinimum, XMIN, XMAX, delta);

                currentValue = random_variate(searchLocalMinimum, XMIN, XMAX, 2);

                if (acceptanceProbability(energy(searchLocalMinimum), energy(currentValue), Temperature) > rand.NextDouble())
                {
                    searchLocalMinimum = currentValue; 
                    ct2++;
                }

                if (energy(searchLocalMinimum) < energy(searcheGlobalMinimum)) 
                {
                    searcheGlobalMinimum = searchLocalMinimum; 
                    ct0++; 
                }

                Temperature *= 1 - coolingRate;
                ct3++;
            }

            return searcheGlobalMinimum;
        }

        static double myFunction(double x)
        {
            return 0.11 * (0.5 * x * x + Math.Cos(Math.PI * x) - 2 * Math.Sin(2 * Math.PI * x) + Math.Cos(3 * Math.PI * x) * Math.Sin(Math.PI * x));
        }

        static void Main(string[] args)
        {
// }

            // COEF is the Most important value to adjust. 
            // energy = COEF x myFunction(x)
            // Change this value until having a quality results for Simulated Annealing. 
            COEF = 1; // <- Try with 1, 10, 100, 1000.....

            // After running the playground :
            // When the ct0 = ct1 or ct2 = ct3, its a bad adjustement.
            // A possible good situation was where ct2 =~ 2 x ct1 and ct3 =~ 10 x ct2.
            
            
            
            // The number of simulations is now fixed to 200.
            // Next time we will see how to change the number of simulations.
// { autofold

            List<double> sols = new List<double>();

            Console.WriteLine("The theoric global minimum is -0.711");
            Console.WriteLine("For 200 subdivisions of ]-5; 5[, the theoric step is 0.05");
            Console.WriteLine();

            var best = simulated_annealing();
            Console.WriteLine("COEF = {0}", COEF);
            Console.WriteLine("Found : Xmin={0,6:0.000}.", best);
            Console.WriteLine("ct0 : {0,4} Global better value found", ct0);
            Console.WriteLine("ct1 : {0,4} Local better value found", ct1);
            Console.WriteLine("ct2 : {0,4} Jump ahead", ct2);
            Console.WriteLine("ct3 : {0,4} Total loops", ct3);
            Console.WriteLine();

            for (int k = 0; k < 100; k++)
            {
                var best0 = simulated_annealing();
                sols.Add(best0);
            }

            Console.WriteLine("Result for 100 repetitions.");
            var avg = sols.Average();
            var std = Math.Sqrt(sols.Sum(s => (s - avg) * (s - avg)) / sols.Count);
            var min = sols.Min();
            var max = sols.Max();
            Console.WriteLine("Avg : Xmin={0,6:0.000}; sigma : {1,6:0.000}", avg, std);
            Console.WriteLine("{0} values are in ] avg - sigma; avg + sigma [", sols.Count(s => s > avg - std && s < avg + std));
            Console.WriteLine("Min : {0,6:0.000}; Max : {1,6:0.000}; Max - Min = {2:0.000}", min, max, max - min);
        }
    }
}
// }
```
